# Lecture 2: Implementation in Java (Calculator) + Classification, Generalization, Specialization

**Date:** Tue 6 Oct 2026 · **Book:** *Object-Oriented Programming and Java*, 2nd ed.: Ch.4 Implementation in Java (pp.39–49) and Ch.5 Classification, Generalization, and Specialization (pp.51–58)

**Time plan (1 h):** 10 min recap (3 bullets from Lecture 1, then answer Lecture 1 self-check Q5 from memory) · 20 min read this lecture + book pages · 30 min hands-on (exercises below).

**Goal for today:** (a) follow a whole class from source file → `javac` → `java` → running objects, and see why the "engine" should be kept apart from the "user interface"; (b) be able to sort objects into classes, arrange those classes into a hierarchy, and say which classes are abstract and which are concrete.

> **Pacing note:** CURRICULUM.md schedules ch.4 and ch.5 together on **Tue** (this file). ch.4 is mostly one worked example; ch.5 is mostly vocabulary and hierarchy design. If the hour runs out, do exercises 1 and 4 today and finish 2 and 3 on **Wed** (book ch.3 deep-dive day). §4.3–4.4 use AWT, a GUI library. Ch.13 (GUI) is **skipped** in this lab, so read §4.4 only for the *event-driven* idea and don't study the AWT API.

---

## Part A: Ch.4 Implementation in Java

### A1. The CalculatorEngine (§4.1)

The book builds a four-function calculator. It starts with the **engine** (the part that calculates) and leaves the keypad and display for later. This is **abstraction**: deal with what matters first and leave the details for later.

How the authors design it, step by step:

1. **What state does a calculator need?** It needs two **registers**, because binary operations have two operands:
   - `value`: what's on the display right now (the number being typed, or the last result).
   - `keep`: the first operand, saved away while you type the second one.
   - Later they add `toDo` (a `char`): which operation to apply when `=` is pressed.
2. **What messages should it accept?** One method per button: `digit(int)`, `add()`, `subtract()`, `multiply()`, `divide()`, `compute()` (the `=` key), `clear()` (the `C` key), plus `display()` to read the result.
3. **How does it start in a valid state?** The constructor simply calls `clear()`.

The book's skeleton (Listing 4-1, abbreviated):

```java
class CalculatorEngine {
    int value;
    int keep;          // two calculator registers
    char toDo;         // pending operation (added in §4.1.4)

    void digit(int x)  { ... }   // value = value*10 + x  (shifts digits left)
    void add()         { ... }
    void subtract()    { ... }
    void multiply()    { ... }
    void divide()      { ... }
    void compute()     { ... }   // the "=" key
    void clear()       { ... }
    int  display()     { ... }

    CalculatorEngine() { clear(); }
}
```

Ideas to take from §4.1.1–4.1.4:
- **`digit(x)` builds a number one key at a time.** Typing `1` then `3` gives `13` because each new digit shifts the old ones one place left. (You can wrap it as `one()`, `two()`, …, each calling `digit(n)`.)
- **Infix input needs memory.** In `1 3 + 1 1 =`, the `+` is pressed *before* the second operand exists. So `add()` can't add yet. It **stashes** the first operand in `keep`, resets `value`, and records `'+'` in `toDo`. The real work happens later in `compute()`.
- **Remove duplication with a helper.** The four operator methods do the same three steps, so the book moves them into `binaryOperation(char op)`, and `add()` becomes a one-liner that calls it. This is **abstraction inside a class**: one private-ish helper, several public messages.
- **`compute()` dispatches on `toDo`** with an `if / else if` chain, then resets `keep`.

> Read the full listing (4-2) in the book. Don't copy it into your repo blind. Exercise 1 asks you to **type it yourself and trace it**.

### A2. Code execution: where does a program start? (§4.2)

- `new CalculatorEngine()` creates an instance; `c.digit(1); c.digit(3); c.add(); …` sends it messages.
- **The chicken-and-egg problem:** objects only run when another object sends them a message. So when the program starts and no objects exist yet, who sends the first one?
- **Answer: a class method.** `public static void main(String[] args)` belongs to the *class* and needs no object. The JVM calls it first, and `main` creates the first objects.
- `System.out.println(...)` is a **Java API** call: library code you use without writing it.
- Compile and run from the terminal:

  ```text
  $ javac CalculatorEngine.java     # source → bytecode (CalculatorEngine.class)
  $ java CalculatorEngine           # JVM loads the class and calls its static main
  ```

- **Modern footnote:** since Java 11 you can run a single source file directly with `java CalculatorEngine.java` (JEP 330). That's handy for exercises, but you should still know the two-step `javac`/`java` flow, because Maven runs it for you in later weeks.
- **Style footnote:** the book writes `String arg[]` (C-style). It's legal, but modern Java style is `String[] args`.

### A3. A simple user interface: separation of concerns (§4.3)

Hard-coding key presses in `main` means editing and recompiling for every new sum. The fix is a **second object** whose only job is the conversation with the user:

- The book's **`CalculatorInput`** (the text calls it "CalculatorInterface"; the listing names it `CalculatorInput`, so don't be confused) holds a reference to an engine. It loops: show `[value]`, read a line, look at the first character, and send the matching message to the engine (`+` → `engine.add()`, a digit → `engine.digit(...)`, `=` → `engine.compute()`, `c` → `engine.clear()`).
- The engine has **no idea** a keyboard exists. The interface has **no idea** how arithmetic is done. Each class has one concern.
- Two parts you're told to "take on faith" for now:
  - `throws Exception` is covered in ch.9 (Week 3).
  - `BufferedReader` / `readLine()` is covered in ch.10 (Week 3). Today, `Scanner` (from `java.util`) is a fine alternative for reading lines from the console.
- The book also points out a benefit for development: the UI object works as a **test harness**. You can poke the engine interactively while you build it.

Two objects working together, each with its own job, is the whole point of OOP from Lecture 1, now in running code.

### A4. Another interface, and event-driven programming (§4.4)

- Because the engine is independent, the book reuses it **unchanged** behind a windowed front end, `CalculatorFrame`. That's the payoff of separation: **same engine, new UI, no edits to the engine**.
- Skip the AWT details (ch.13 is skipped). Notice only *how control flows*:

| | `CalculatorInput.run()` (procedural) | `CalculatorFrame.actionPerformed()` (event-driven) |
|---|---|---|
| Who calls the dispatch code? | Your own loop, started from `main` | The **framework**, when a button is clicked |
| Who decides the order of execution? | Your code, step by step, like a recipe | The **user**: events arrive in any order |
| How is the code hooked up? | Called directly | **Registered** as a listener (`addActionListener(this)`), then *called back* |

- **Event-driven programming:** you attach pieces of code to **events** (clicks, key presses, window closing). You don't decide when they run. You only decide what happens when they do. This is called **inversion of control**, and it's the mental model behind Spring (Week 3+): Spring calls *your* controller method when an HTTP request arrives, much like AWT calls `actionPerformed` on a click.
- **Dated APIs in Listing 4-4:** `new Integer(...)` is deprecated (use `String.valueOf(...)` / `Integer.toString(...)`), and `Frame.show()` is deprecated (use `setVisible(true)`). Treat the listing as history.

---

## Part B: Ch.5 Classification, Generalization, Specialization

### B1. Classification (§5.1)

- **Classification** means spotting objects that share properties and grouping them into a **class**. The book takes 18 named animals (Mighty the elephant, Swift the eagle, Jaws the shark, …) and groups them by shared traits. Mammals: born alive, warm-blooded, lungs, hair. Birds: beak, two legs, wings, feathers, lay eggs. That gives Mammal, Bird, Fish, Reptile, Insect and Amphibian.
- A class is a **meaningful abstraction**: when you talk about "the Bird class", you are indirectly talking about all its objects.

### B2. Superclass, subclass, and the hierarchy diagram (§5.2)

- HomeCare example: Sean and Sara are `SalesPerson`s, Simon and Sandy are `Manager`s, and **all four are `Employee`s**. An object can belong to several classes at once, from general to specific.
- Calling Sean an *Employee* is **general**: you ignore what makes him different. Calling him a *SalesPerson* is **specific**: he takes orders and earns commission, which a Manager doesn't.
- **Superclass** = the more general class; **subclass** = the more specialized class. In a **class hierarchy diagram**, general classes go at the top, specialized ones at the bottom, and a triangle/arrow points **up to the superclass**.
- A class can be **both** a superclass and a subclass. `Employee` is a subclass of `Person` and a superclass of `Manager` and `SalesPerson`.

### B3. Generalization vs specialization (§5.3–5.4)

| | Generalization | Specialization |
|---|---|---|
| Direction | **Bottom-up**: from existing classes to a new, more general parent | **Top-down**: from a class to new, more specific children |
| What you capture | **Similarities** between classes | **Differences** between objects of one class |
| Book example | Mammal, Fish, Bird, Reptile, Amphibian all have a backbone → new superclass *Animal-with-Backbone*; then both backbone groups → *Animal* | *Animal* split into with- and without-backbone; *Animal-with-Backbone* split into Mammal, Fish, … |

**Rule of organization (§5.5):** the higher you go, the more general the class and the **more** objects fit in it. The lower you go, the more specialized the class and the **fewer** objects fit.

### B4. Abstract vs concrete classes (§5.6)

- **Abstract class:** so general that you never intend to create objects from it. It exists to hold what its subclasses have in common. Examples: *Animal*, *Animal-with-Backbone*.
- **Concrete class:** a class you actually instantiate. In the book these are the leaves: Mammal, Fish, Bird, ….
- In Java you mark it with the `abstract` keyword. The compiler then **refuses** `new` on it:

```java
abstract class Vehicle { }                 // illustrative domain, not the exercise
abstract class MotorVehicle extends Vehicle { }
class Motorbike extends MotorVehicle { }   // concrete

Vehicle v = new Vehicle();     // compile error: Vehicle is abstract; cannot be instantiated
Motorbike m = new Motorbike(); // fine
```

- `extends` shows up here only as notation. **How** subclasses inherit fields and methods is ch.6 (Week 2, Monday). Abstract *methods* (methods with no body that subclasses must supply) also come with ch.6–7.
- **The book's Java here isn't legal as printed:** `Animal-with-Backbone` contains hyphens, which aren't allowed in Java identifiers. A real name would be `VertebrateAnimal` or `AnimalWithBackbone`. The listing also repeats `Mammal` and `Fish` twice (a typesetting slip).

---

## Glossary

**Ch.4:** register (state field) · engine vs user interface · separation of concerns · abstraction · helper method (`binaryOperation`) · infix notation · dispatch (choosing which method to call based on input) · `static` / class method · `main` (entry point) · Java API · `javac` (compiler) · `java` (launcher/JVM) · bytecode / `.class` file · test harness · procedural programming · event-driven programming · event · listener / callback · inversion of control

**Ch.5:** classification · class hierarchy · superclass · subclass · class hierarchy diagram · generalization · specialization · abstract class · concrete class · `abstract` keyword · `extends` (preview)

## Common pitfalls

1. **Putting I/O inside the engine.** The moment `CalculatorEngine` calls `System.out` or reads the keyboard, you can't reuse it behind another UI or test it easily. Keep the engine silent: return values and let the UI print.
2. **Forgetting that `main` is `static`.** Inside `main` there is no `this`. Calling an instance method or field directly from `main` gives *"non-static … cannot be referenced from a static context"*. Create an object first.
3. **File name ≠ public class name.** A `public class Foo` must live in `Foo.java`. With `java Foo` you give the **class** name (no `.class`). With single-file launch (`java Foo.java`) you give the **file**.
4. **Integer division and division by zero.** `int / int` truncates, and `x / 0` on `int`s throws `ArithmeticException` at runtime. The book's engine has no protection here. You decide in exercise 2 what *your* engine should do.
5. **Uninitialized `char toDo`.** A `char` field defaults to `'\u0000'`, which matches none of `+ - * /`. Think about what `compute()` does if `=` is pressed before any operator.
6. **Comparing `char` with a `String`.** `'+'` (char, single quotes) is not `"+"` (String, double quotes). `m.charAt(0) == "+"` doesn't compile.
7. **Treating every difference as a new subclass.** If two kinds of objects differ only in **data** (colour, price, name), that's usually a **field** (or an `enum`), not a subclass. Subclass when **behaviour or structure** differs.
8. **Deep hierarchies "because the real world has them".** Model what the *program* needs, not the whole of biology. 2–3 levels is usually plenty.
9. **Trying to `new` an abstract class**, or forgetting to mark a purely general class `abstract`, so that someone can create a meaningless "generic Animal".
10. **Mixing up the directions.** Generalization goes *up* (find what's common), specialization goes *down* (find what's different), and subclass arrows point *up* to the superclass.
11. **Copying deprecated book APIs** (`new Integer(…)`, `show()`) into new code. Check the Java 21 API docs when a book API looks odd.

## Class vs function, applied to today's material

Lecture 1's decision questions (state? rules to protect? just data? several implementations?) applied to today's examples:

- **Why is `CalculatorEngine` a class?** Because it has to **remember between calls**. Each key press is a separate message, and the engine must keep `value`, `keep` and `toDo` from one press to the next. A pure function would have nowhere to put that memory.
- **When would a function be enough?** If the input were a *complete* expression handed over all at once, such as `(13, '+', 11)` or the string `"13+11"`, evaluating it needs no memory between calls. That's a **static function**: inputs in, result out, easy to test. Same domain, different shape of input, different right answer.
- **Why is the UI a separate class, not more methods on the engine?** It has its own state (the input stream, a reference to the engine) and a different **reason to change**: UIs change often, arithmetic rarely does. Different reasons to change → different classes.
- **Why is the listener an object?** In event-driven code, the framework needs something it can *hold on to* and *call later*. In 2008 Java that had to be an object implementing an interface. In Week 2 you'll see that a **lambda** can stand in for a one-method listener: functions-as-values meeting objects.
- **Hierarchies are about classes with behaviour.** Classification, generalization and specialization organize **classes**. A family of static functions doesn't need a hierarchy. If you catch yourself designing an abstract class that has no state and no behaviour of its own, it's probably just a namespace for functions, and an abstract class is the wrong tool.

---

## Exercises (write the code yourself; no solutions here)

Put your work in `exercises/2026-10-06-ch4-5/`. Compile and run each one, and commit when it works. Exercises 2 and 4 are warm-ups for CURRICULUM Week 1 exercises 3 and 4.

1. **Trace, then type.** Before writing any code, draw a table in `notes/2026-10-06-trace.md` with columns *key pressed · `value` · `keep` · `toDo`*, and fill it in **by hand** for these key sequences on the book's engine: `1 3 + 1 1 =` · `9 - 4 = =` · `5 + =` · `8 / 0 =` · `= 7`. Then type Listing 4-2 yourself (don't paste it), compile it with `javac`, run it with `java`, and change `main` to check each prediction. Mark each row where your prediction was wrong and write one sentence on why.

2. **Grow the engine without touching the UI's job.** Extend your own copy of the engine with two **unary** operations (one operand, e.g. square and factorial, or your own choice). Before coding, write in `notes/` how a unary operation differs from a binary one in what it does to `value`, `keep` and `toDo`. Also decide and document a policy for division by zero (what the user sees, and what state the engine is left in), and implement it. Then add keys for the new operations to a line-mode front end you write yourself (`Scanner` is fine). Constraint: the engine class must contain **no** `System.out` and **no** input reading.

3. **Event-driven without a GUI.** Without using AWT or Swing, simulate event-driven control. Write a small class that represents a *button*: it has a label and a reference to the engine it notifies, and when it's "pressed" it sends the right message to the engine. In `main`, build a set of buttons, then press them in an order that **`main` doesn't hard-code**: take it from the command-line arguments, or from a line the user types. In `notes/`, answer: in your program, who decides the order in which engine methods run, and how is that different from exercise 1's `main`?

4. **Classify a domain of your own.** Choose a domain from your own work or life (not animals, not employees, not vehicles). List at least 10 concrete objects, each with a few properties and behaviours, like Table 5-1. In `notes/2026-10-06-classification.md`: (a) classify them into classes; (b) **generalize** upward at least two levels, naming each new superclass and the similarity it captures; (c) mark every class *abstract* or *concrete*; (d) for one place where you were tempted to make a subclass, explain why a field would or wouldn't do instead. Then write empty-bodied Java classes for the hierarchy using `abstract` and `extends`, get them compiling, and add one line to `main` that tries to instantiate an abstract class. Paste the compiler's error message into your notes, then comment the line out.

*(Optional warm-up if time remains: do book §5.8 Q2–Q4 on paper, classifying Aaron the ant through Angel the goldfish.)*

## Self-check (answer in `notes/`, then check against the book)

1. Why does `CalculatorEngine` need `keep` and `toDo` as well as `value`? What would go wrong in `1 3 + 1 1 =` with only one register?
2. At program start no objects exist. How does the first object get created, and why must `main` be `static`?
3. Compare `CalculatorInput.run()` and `CalculatorFrame.actionPerformed()`: who calls each, who controls the order of execution, and which one is event-driven?
4. Define generalization and specialization in your own words, give one example of each from outside the book, and say which direction each moves in a class hierarchy diagram.
5. What makes a class abstract rather than concrete? Why does `new Animal()` fail to compile in the book's example, and why can't `Animal-with-Backbone` be a Java class name as printed?

*Bonus (book §4.6 Q4, no answer given here):* on the book's calculator, the key sequence `1 3 + 1 1 = 7` displays `247`. Work out why from the engine's state, and describe (in words, not code) what the engine would have to remember to behave like a real calculator.

*Stuck for more than 15 min? Write the exact error/question in `notes/` and ask for a review. You'll get hints and feedback, not solutions.*
