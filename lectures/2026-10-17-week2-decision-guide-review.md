# Lecture 8 — Week 2 review: class vs record vs interface vs static function

**Date:** Sat 17 Oct 2026 · **Book:** none today (recap ch.6–8 + Fri modern track per `CURRICULUM.md` Week 2)

**Time plan (1 h):** 10 min recap (whole week from memory) · 20 min read this lecture + your week notes · 30 min hands-on (see §12 — curriculum **#3–#5** primary; finish **#1–#2** if still open; problem statements in `CURRICULUM.md` only).

**Goal for today:** close Week 2 with a **decision guide** you can reuse in later weeks, finish or polish the **five proof exercises**, and rehearse when to reach for a **static function**, **record**, **class**, **interface**, **enum**, or **sealed interface + records** — without adding new book chapters or schedule shifts.

> **Pacing note:** Week 2 in `CURRICULUM.md` schedules **Mon = ch.6 §6.1–6.7**, **Tue = §6.8 interfaces**, **Wed = ch.7**, **Thu = ch.8**, **Fri = lambdas & streams**, **Sat = decision guide + review** (this file). Saturday is the **lighter morning** slot in the curriculum cadence, but it is still a full **1-hour block** — review and consolidation, not a day off. Do not invent schedule changes inside this lecture.

---

## 1. Recap hook (Week 2 — use the first 10 minutes)

Without opening the book or old lectures, in `notes/`:

1. **Three bullets from yesterday (Fri):** e.g. a **functional interface** has one abstract method; streams are **source → intermediate* → terminal**; curriculum **#2** asks for the **same three questions** with streams **and** loops.
2. **One self-check from Lecture 7:** e.g. why you must not **reuse** a stream after a terminal operation.
3. **Skim** your compile/test output from **#1** / **#2** — note one error you fixed or one question still open.

Then add **three more bullets** for the **whole week** (Mon–Thu), one theme each: **inheritance & `super`**, **interfaces & binding**, **polymorphism**, **packages & visibility**. If any day was missed, do **not** double up two book chapters today — note the gap in `notes/` and continue with hands-on + proof artifact (curriculum weekly rhythm).

*(Book ch.13–15 — GUI, applets, servlets — remain **skipped**; see `CURRICULUM.md` [Book chapters skipped on purpose](#book-chapters-skipped-on-purpose).)*

---

## 2. Week 2 arc (what you were building toward)

| Day | Theme (curriculum) | Lecture file | Core ideas to carry into today |
|---|---|---|---|
| **Mon 12** | ch.6 §6.1–6.7 inheritance | `lectures/2026-10-12-ch6-inheritance.md` | `extends`, `super`, `@Override`, *is-a* vs *has-a* |
| **Tue 13** | ch.6 §6.8 interfaces | `lectures/2026-10-13-ch6-interfaces.md` | contracts, default/static interface methods, functional interfaces |
| **Wed 14** | ch.7 polymorphism | `lectures/2026-10-14-ch7-polymorphism.md` | static vs dynamic binding, overload vs override |
| **Thu 15** | ch.8 modularity | `lectures/2026-10-15-ch8-modularity.md` | packages, `import`, visibility, encapsulation trade-offs |
| **Fri 16** | lambdas & streams | `lectures/2026-10-16-lambdas-streams.md` | lambdas, method refs, `Optional`, `filter`/`map`/`collect`/`reduce`, sealed preview |
| **Sat 17** | decision guide + review | *this file* | choose the right construct; proof **`week2`** |

**Book read this week (paraphrase only in your notes):** Ch.6 Inheritance (pp.61–90), Ch.7 Polymorphism (pp.93–102), Ch.8 Modularity (pp.103–117).

**Modern thread (not in the 2008 book):** lambdas, streams, sealed types — aligned with `CURRICULUM.md` Week 2 **Topics** and **Free resources**.

---

## 3. Decision guide: what to reach for (anchor for weeks 1–2)

The table below **paraphrases** the decision guide in **`CURRICULUM.md` Week 2** ([Decision guide: class vs record vs interface vs static function](#decision-guide-class-vs-record-vs-interface-vs-static-function)). Your proof artifact includes **`notes/class-vs-record-vs-interface.md`** in **your own words** — same rows, your examples from **#1–#5**, not a copy-paste of this lecture.

| Reach for… | When | Typical example |
|---|---|---|
| **Static function** | Pure computation: output depends only on inputs, no state, no polymorphism needed | `Math.max`, `TaxCalculator.vat(amount)` |
| **Record** | Immutable data carrier; equality by value; few/no behaviours beyond validation | `record Point(int x, int y)`, DTOs, API responses |
| **Class** | Mutable state with invariants to protect, or identity matters, or a lifecycle | `BankAccount`, `Connection`, JPA entities (later weeks) |
| **Interface** | A *contract* with ≥2 plausible implementations, or swap/mock behaviours | `PaymentMethod`, `Repository`, `Comparator` |
| **Enum** | Fixed, known set of constants (possibly with behaviour) | `OrderStatus` |
| **Sealed interface + records** | Closed set of *variants*; compiler-checked exhaustive `switch` | `Shape` = `Circle`, `Rect`, `Triangle` |

### 3.1 How the week’s topics map to the guide

| Construct | Where you practiced it |
|---|---|
| **Class + inheritance** | Mon hierarchy; **#4** refactor source |
| **Interface + polymorphism** | Tue **#1** “payable”; Wed binding table |
| **Package + class visibility** | Thu **#2** order model in named packages |
| **Record + streams** | Fri **#2** immutable `Order`-style rows; method references |
| **Sealed + records + switch** | **#3** closed shape set |
| **Composition vs inheritance** | **#4** |
| **Label every type in a tiny app** | **#5** |

### 3.2 Quick decision flow (mental checklist)

Use this when **#5** or the proof guide asks you to label a type:

1. **Fixed small set of named constants?** → **enum**.
2. **Closed set of variants + exhaustive handling?** → **sealed interface** + **records** (or sealed class hierarchy).
3. **Multiple implementations of the same behaviour contract?** → **interface** (implementation classes or lambdas where SAM fits).
4. **Only data, immutable, value equality?** → **record**.
5. **Mutable state, invariants, or object identity over time?** → **class**.
6. **Stateless pure helper, no polymorphism?** → **static function** (often in a small `util` class or the type that owns the domain).

If two rows seem to fit, write **one sentence** in `notes/class-vs-record-vs-interface.md` explaining which you chose and what you gave up.

### 3.3 Common confusions (review, not new material)

| Confusion | Clarification |
|---|---|
| “I used a class for data only.” | Week 1 **#5** vs **record** — list what you lost/gained; Sat guide should say when **record** is enough. |
| “Interface vs abstract class.” | Interface = **multiple types**, **has-a** contracts, functional SAM; abstract class = shared **implementation** + single inheritance **is-a** (book ch.6–7). |
| “Static method on interface vs static utility.” | Both can be stateless; interface **static** ties to the type’s contract; package **util** when no implementing type story. |
| “Stream/lambda replaced OOP.” | Lambdas **implement** functional interfaces; **#2** still needs a sensible **model** (often **records**) and maybe a **service** **class** for orchestration. |

---

## 4. Curriculum exercises — status check (no solution code here)

Full wording of **#1–#5** lives in **`CURRICULUM.md` Week 2** **Hands-on exercise prompts** only. Today’s priority:

| Prompt | Sat role |
|---|---|
| **#1** | Finish if open — binding explanation in `notes/` is part of the proof story |
| **#2** | Finish if open — **both** loop and stream versions + readability paragraph |
| **#3** | **Primary hands-on** — sealed shapes, records, area via exhaustive `switch` |
| **#4** | **Primary hands-on** — composition refactor + 5-sentence justification |
| **#5** | **Primary hands-on** + feeds **`notes/class-vs-record-vs-interface.md`** |

**Re-attempt rule (curriculum weekly rhythm):** if one exercise was hard, redo the **hardest part from a blank scratch file** for 10–15 min inside today’s block before tagging — quality over rushing all five folders.

---

## 5. Proof artifact (end of today)

Per **`CURRICULUM.md` Week 2**:

1. **`exercises/week2/`** — all **five** exercises, each compiling and runnable (one-line run hint in a comment or local README comment).
2. **`notes/class-vs-record-vs-interface.md`** — one-page decision guide in **your words**, anchored on the curriculum table (§3 above cites the same source).
3. **Git commit tagged `week2`** — no secrets; no `.env`.

Optional consolidation: copy or move scratch from `exercises/2026-10-12-ch6/` … `exercises/2026-10-16-lambdas-streams/` into **`exercises/week2/`** with clear package names.

Closing habit (2 min): one line in `notes/` — what clicked / what is still fuzzy going into Week 3.

---

## 6. Week 2 glossary (cold recall)

inheritance · `extends` · `super` · override · overload · interface · default method · functional interface · SAM · polymorphism · static binding · dynamic binding · package · package-private · encapsulation · lambda · method reference · stream · `filter` · `map` · `collect` · `reduce` · sealed · permits · exhaustive switch · composition · aggregation · record · enum · static utility

---

## 7. Self-check (answer in `notes/`, then verify against week lectures)

1. When does **dynamic binding** choose the method implementation?
2. Why can a **subclass in another package** not access a **package-private** superclass member?
3. Name one case where a **loop** was clearer than a **stream** for curriculum **#2** (from your own notes).
4. What makes a **sealed interface** + **records** a good fit for **#3**?
5. In **#4**, what relationship did you change from inheritance to composition — and why?
6. For **#5**, list one type you labeled **interface** and one **static function** with your one-line reason.

*Stuck for more than 15 min? Write the exact error/question in `notes/` and ask for a review — hints and feedback, not solutions.*

---

## 8. Bridge to Week 3

Next week per **`CURRICULUM.md`**: **exceptions**, **I/O**, and the **first Spring Boot REST endpoint** (book ch.9–10; **ch.13–15 still skipped**). Your Week 2 decision guide applies immediately: **records** for JSON DTOs, **classes** for services with lifecycle, **interfaces** for test doubles, **static utilities** for pure parsers.

---

## 9. Free resources (optional Sat skim — Week 2 list only)

If a self-check failed, revisit links from **`CURRICULUM.md` Week 2 Free resources** (Oracle *Interfaces and Inheritance*, *Lambda Expressions*, *Aggregate Operations*, dev.java Streams/Lambdas/Sealed classes, Baeldung streams and composition). No new URLs invented here.

---

## 10. Common pitfalls (Week 2 review)

1. **Proof folder empty** — Sat is for **`exercises/week2/`**, not only scratch dated folders.
2. **Decision guide missing** — **`notes/class-vs-record-vs-interface.md`** is required for tag **`week2`**, not optional commentary.
3. **`default` branch on sealed `switch` for #3** — curriculum asks for exhaustiveness **without** `default` when the permitted set is complete.
4. **#2 only streams** — curriculum requires **loops too** and comparison in `notes/`.
5. **#1 binding explanation missing** — demonstrate output + explain static vs dynamic in **your** notes.
6. **Public everything in `exercises/week2/`** — apply Thu ch.8: narrow **public** API, hide implementation.
7. **Skipping #5** — it is the typed checklist for the decision guide; do it even if the “tiny app” is minimal.
8. **Inventing Week 3 reading today** — stay on Week 2 proof; open **`CURRICULUM.md` Week 3** only for curiosity after the tag.

---

## 11. Hands-on (Sat — curriculum **#3–#5** primary; you write all code)

**Reading (20 min slot):** this lecture §2–§3; skim your Mon–Fri `notes/` and **`CURRICULUM.md` Week 2** decision table; draft or refine **`notes/class-vs-record-vs-interface.md`** (outline is fine before coding).

**Primary Sat focus** (per `CURRICULUM.md` Week 2 **Sat = decision guide + review**):

### Curriculum **#3** (problem statement only)

Model a **closed set of shapes** using a **sealed interface** and **records**; write a function that computes **area** using a **`switch`** over the shape types with **no `default`** branch.

### Curriculum **#4** (problem statement only)

Take a **class hierarchy of 3 levels** that you wrote in **week 1** and refactor it to **prefer composition over inheritance** for one relationship. Justify the choice in **5 sentences** (in `notes/` or beside the exercise).

### Curriculum **#5** (problem statement only)

For a **tiny app** of your choice (e.g. a to-do list), list **every type** you need and label each ***class / record / interface / enum / static utility function*** with a **one-line reason**.

| Other prompts | Sat guidance |
|---|---|
| **#1** | Complete if unfinished before or after **#3–#5**; ensures interface + binding story in proof set |
| **#2** | Complete if unfinished; bring Fri loop vs stream notes into **`class-vs-record-vs-interface.md`** |

Work under `exercises/week2/` (preferred for proof), or finish scratch then **consolidate** into `exercises/week2/` before the **`week2`** tag.

When all five compile/run and the decision guide file exists: commit and tag **`week2`**.

---

## 12. Saturday tone reminder

Lighter **does not mean shorter**: one focused hour on **choosing constructs deliberately** and **shipping the proof artifact** beats rereading an entire chapter. If everything is green early, spend remaining minutes on **§7 self-check** cold or a **10-minute re-attempt** of your hardest exercise — then stop; **Sunday is rest** per curriculum rhythm.
