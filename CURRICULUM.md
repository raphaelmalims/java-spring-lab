# CURRICULUM — Java refresher → Spring Boot → Java ML/AI systems

**Learner:** Raphael Malims · **Start:** Mon 5 Oct 2026 · **Cadence:** 1 h/day, Mon–Sat (Sat = longer-feeling "build & review" day, still 1 h) · **Length:** 10 weeks + 1 buffer week (14–19 Dec 2026)

**Ground rules**
- Raphael writes **all** exercise code himself. Exercise prompts below are *problem statements only*; no solutions are provided by agents. Agents write lectures, notes and reviews only.
- Book = *Object-Oriented Programming and Java*, 2nd ed. (Poo, Kiong, Ashok, 2008). Page numbers below are the **printed** page numbers from its table of contents. The book dates from 2008, so it lacks modern Java, so each week adds modern Java (records, `var`, lambdas, streams, sealed types, virtual threads, `java.util.concurrent`), JVM memory and Spring Boot from free sources.
- Never commit secrets or `.env` files.
- Target JDK: **Java 21 LTS** (or newer LTS if available). Build tool: Maven (Gradle optional). IDE: IntelliJ IDEA Community or VS Code + Java extensions.

### Book chapters skipped on purpose

These chapters are **not** scheduled in the weekly plan (time goes to modern topics already listed: collections/streams depth, JDBC ch.17, JPA, Spring Boot, buffer-week catch-up).

| Chapter | Topic | Why skip |
|---|---|---|
| **Ch.13** | Graphical interfaces (AWT/Swing) | Dated desktop UI; the lab targets APIs, JVM, and Spring — not Swing apps. |
| **Ch.14** | Applets | Removed/deprecated in modern JDKs; not applicable to current Java. |
| **Ch.15** | Servlets | Raw servlet programming; **Spring Boot** (from Week 3 onward) is the web path instead of hand-written servlets. |

**Still in the plan (book):** **Ch.16** Serialization and RMI — **skim only** in Week 4 (know that `Serializable` exists; treat RMI as legacy; prefer HTTP/JSON and Spring). **Ch.17** JDBC — read in Week 4 alongside Spring Data JPA so you see what JPA abstracts (drivers, `Connection`, prepared statements, try-with-resources).

## Daily 1-hour block template

| Minutes | Activity | Notes |
|---|---|---|
| 0–10 | **Recap** | Without looking: write 3 bullets from yesterday + answer 1 self-check question from the previous lecture. Skim your last compile/test error. |
| 10–30 | **Lecture read** | Read the day's lecture in `lectures/` and the assigned book pages / doc links. Mark 2 things you did not understand. |
| 30–60 | **Hands-on** | Do the day's exercise prompt(s) in `exercises/` **by typing the code yourself**. Compile and run. Commit when it works (or note exactly where you are stuck in `notes/`). |

Closing habit (2 min inside the hands-on slot): add a one-line "what I learned / what confused me" to `notes/`.

## Resource key (all free)

- **Oracle Tutorials:** https://docs.oracle.com/javase/tutorial/
- **dev.java (official Java learning site):** https://dev.java/learn/
- **Java SE API docs:** https://docs.oracle.com/en/java/javase/21/docs/api/
- **OpenJDK JEPs:** https://openjdk.org/jeps/
- **Spring Boot reference:** https://docs.spring.io/spring-boot/
- **Spring guides:** https://spring.io/guides
- **Spring Data JPA reference:** https://docs.spring.io/spring-data/jpa/reference/
- **Spring AI reference:** https://docs.spring.io/spring-ai/reference/
- **Baeldung:** https://www.baeldung.com/ (search by topic)
- **Inside Java / JVM docs:** https://docs.oracle.com/en/java/javase/21/gctuning/ (GC tuning guide), https://docs.oracle.com/en/java/javase/21/troubleshoot/

## Schedule overview

| Week | Dates (2026) | Theme |
|---|---|---|
| 1 | 5–10 Oct | Core Java I: OOP, classes, quick tour, modern basics (`var`, records) |
| 2 | 12–17 Oct | Core Java II: inheritance, interfaces, polymorphism, modularity + lambdas & streams; class vs record vs interface vs static function |
| 3 | 19–24 Oct | Exceptions, I/O + first Spring Boot REST endpoint |
| 4 | 26–31 Oct | Generics, collections internals + JPA basics |
| 5 | 2–7 Nov | JVM memory: stack, heap, GC |
| 6 | 9–14 Nov | References, leaks, profiling, JUnit testing, configuration |
| 7 | 16–21 Nov | Concurrency I: threads (ch.11) → `java.util.concurrent` |
| 8 | 23–28 Nov | Concurrency II: CompletableFuture, virtual threads, pitfalls + calling an LLM from Spring |
| 9 | 30 Nov–5 Dec | Project: Spring service in front of the PersonaLearn RAG endpoint (build) |
| 10 | 7–12 Dec | Project: harden, test, document, demo |
| Buffer | 14–19 Dec | Catch-up, revisit weak spots, polish repo |

---

## Week 1 (5–10 Oct) — Core Java I: objects, classes, quick tour, modern basics

**Topics:** OOP mindset; objects, classes, messages, methods, client/server roles; primitive types; fields, methods, constructors, overloading; control flow, arrays; first implementation walk-through (Calculator); classification, generalization, specialization, abstract vs concrete; **modern:** `var`, `record` basics, `enum`, text blocks, `switch` expressions; class vs function primer.

**Book:** Ch.1 Introduction (pp.1–5), Ch.2 Object, Class, Message and Method (pp.7–14), Ch.3 A Quick Tour of Java (pp.17–36), Ch.4 Implementation in Java (pp.39–49), Ch.5 Classification, Generalization, Specialization (pp.51–58).

**Daily split (suggested):** Mon ch.1–3 (see `lectures/2026-10-05-ch1-3-oop-basics.md`) · Tue ch.3 deep-dive (types, constructors, overloading) · Wed ch.4 · Thu ch.5 · Fri modern basics (`var`, records, enums, switch expressions) · Sat review & mini-project.

**Free resources:**
- Oracle Tutorials: *Learning the Java Language* (Object-Oriented Programming Concepts, Language Basics, Classes and Objects, Numbers and Strings, Enum Types).
- dev.java: *Learn Java → The Language* (records, `var`, switch expressions, text blocks).
- JEP 395 (Records), JEP 286 (Local-Variable Type Inference), JEP 361 (Switch Expressions).
- Baeldung: "Java Records Keyword", "Java 10 Local Variable Type Inference", "Java Constructors".

**Hands-on exercise prompts (problem statements only):**
1. Model a bank account as a class: it must hold an owner name and balance, allow deposits and withdrawals, and refuse operations that would break its rules. Decide yourself what is public, what is private, and what happens on invalid input; write a `main` that exercises it.
2. Write a class that counts how many instances of itself have been created, and a second variant that produces unique sequential IDs. Explain in `notes/` why this state belongs on the class and not on instances.
3. Take the book's Calculator idea (ch.4) and design your own text-based version: separate the "engine" from the "user interface" so the engine can be driven by different front-ends. Support at least four operations and a `clear`.
4. Create a small hierarchy for a domain of your choice (e.g. vehicles, notifications, shapes) with at least one abstract class and two concrete subclasses, then draw the hierarchy as a diagram in `notes/`.
5. Re-express exercise 1's data-only parts using a `record`; list in `notes/` what you gained and what you could no longer do.

**Proof artifact:** `exercises/week1/` containing the five exercises, each compiling and runnable with a one-line `README` comment on how to run it, plus a `notes/week1.md` with 5 bullet "what clicked" and 3 "still fuzzy". Commit tagged `week1`.

---

## Week 2 (12–17 Oct) — Core Java II: inheritance, interfaces, polymorphism, modularity + lambdas/streams

**Topics:** common properties and inheritance; `extends`, `super`, method overriding; changing hierarchies; multiple inheritance problem and **interfaces** (default/static/private interface methods, functional interfaces); static vs dynamic binding; overloading vs overriding; polymorphism; class (static) members; visibility, packages, `import`; encapsulation trade-offs. **Modern:** lambdas, method references, `Optional`, Stream API basics (`filter/map/collect/reduce`), sealed interfaces. **Decision lecture:** *class vs record vs interface vs static function* (see decision guide below).

**Book:** Ch.6 Inheritance (pp.61–90), Ch.7 Polymorphism (pp.93–102), Ch.8 Modularity (pp.103–117).

**Daily split (suggested):** Mon ch.6 §6.1–6.7 · Tue ch.6 §6.8 interfaces · Wed ch.7 · Thu ch.8 · Fri lambdas & streams · Sat decision guide + review.

**Free resources:**
- Oracle Tutorials: *Interfaces and Inheritance*, *Lambda Expressions*, *Packages*, *Controlling Access to Members*, *Aggregate Operations* (streams).
- dev.java: *Learn Java → Streams*, *Lambdas*, *Sealed classes*.
- JEP 409 (Sealed Classes), JEP 441 (Pattern Matching for switch).
- Baeldung: "Guide to Java 8 Streams", "Java 8 Functional Interfaces", "Composition, Aggregation, and Association", "Favor Composition over Inheritance".

**Hands-on exercise prompts:**
1. Define an interface for "something that can be paid" and implement it in at least three unrelated classes. Write a method that accepts the interface type and processes a mixed list. Then demonstrate (with output) which method version runs and explain static vs dynamic binding in your notes.
2. Given a list of at least 10 orders (id, customer, amount, status), answer these using streams *and* again using plain loops: total amount per customer, the top 3 orders by amount, and whether any order is unpaid. Compare readability in `notes/`.
3. Model a closed set of shapes using a sealed interface and records; write a function that computes area using a `switch` over the shape types with no `default` branch.
4. Take a class hierarchy of 3 levels that you wrote in week 1 and refactor it to prefer composition over inheritance for one relationship. Justify the choice in 5 sentences.
5. For a tiny app of your choice (e.g. a to-do list), list every type you need and label each *class / record / interface / enum / static utility function* with a one-line reason.

**Proof artifact:** `exercises/week2/` with the five exercises + a one-page `notes/class-vs-record-vs-interface.md` decision guide in your own words. Commit tagged `week2`.

### Decision guide: class vs record vs interface vs static function (anchor for weeks 1–2)

| Reach for… | When | Typical example |
|---|---|---|
| **Static function** | Pure computation: output depends only on inputs, no state, no polymorphism needed | `Math.max`, `TaxCalculator.vat(amount)` |
| **Record** | Immutable data carrier; equality is by value; few/no behaviours beyond validation | `record Point(int x, int y)`, DTOs, API responses |
| **Class** | Has mutable state with invariants to protect, or identity matters, or a lifecycle | `BankAccount`, `Connection`, JPA entities |
| **Interface** | A *contract* with ≥2 plausible implementations, or you want to swap/mock behaviours | `PaymentMethod`, `Repository`, `Comparator` |
| **Enum** | Fixed, known set of constants (possibly with behaviour) | `OrderStatus` |
| **Sealed interface + records** | Closed set of *variants* you want the compiler to check exhaustively | `Shape` = `Circle`, `Rect` or `Triangle` |

---

## Week 3 (19–24 Oct) — Exceptions, I/O + first Spring Boot REST endpoint

**Topics:** exception terminology; checked vs unchecked; `try/catch/finally`, multi-catch, custom exceptions, exception chaining; **try-with-resources**; the Java API docs; byte vs character streams; files and `java.nio.file` (`Path`, `Files`); `Scanner`, formatting; **Spring Boot:** project via start.spring.io, `@RestController`, request mapping, JSON via records, `ResponseEntity`, `@ExceptionHandler`/`@RestControllerAdvice`.

**Book:** Ch.9 Exception Handling (pp.119–134), Ch.10 Input and Output Operations (pp.135–154; focus §10.1–10.4, 10.7–10.10; skim 10.5–10.6 and 10.11).

**Daily split:** Mon ch.9 §9.1–9.5 · Tue ch.9 §9.6 + try-with-resources · Wed ch.10 streams & files · Thu `java.nio.file` + `Scanner` · Fri Spring Boot "hello REST" · Sat REST + error handling.

**Free resources:**
- Oracle Tutorials: *Essential Java Classes → Exceptions*, *Basic I/O*, *File I/O (NIO.2)*.
- Spring: *Building a RESTful Web Service* guide (spring.io/guides/gs/rest-service), Spring Boot reference → *Web → Servlet Web Applications*, *Error handling*.
- Baeldung: "Exception Handling in Java", "Java try-with-resources", "Guide to java.nio.file", "Spring Boot Error Handling / @ControllerAdvice".

**Hands-on exercise prompts:**
1. Write a program that reads a text file of numbers (some lines intentionally malformed), sums the valid ones, reports each bad line with its line number, and never crashes. Decide where to catch and where to propagate.
2. Define your own checked exception and your own unchecked exception for a domain rule (e.g. insufficient funds, invalid email). Write a short rationale for each choice.
3. Write a utility that copies a file using try-with-resources, then rewrite it with `Files.copy`. Time both on a large file and note differences.
4. Build a Spring Boot app with `GET /api/todos` and `POST /api/todos` storing items in memory; return correct HTTP status codes and a JSON error body for bad input.
5. Add `GET /api/todos/{id}` that returns 404 with a structured error when missing, using centralized exception handling.

**Proof artifact:** Running Spring Boot app in `exercises/week3-todo-api/` with a `curl` transcript (in `notes/week3.md`) showing 201, 200, 400 and 404 responses; plus the file-parsing exercise. Tag `week3`.

---

## Week 4 (26–31 Oct) — Generics, collections internals + JPA basics

**Topics:** why generics (RTTI/casts problem); generic classes and methods; bounded types, wildcards (PECS); type erasure; Collections Framework interfaces; **ArrayList** (resizing, amortized O(1) append), **HashMap/HashSet** (hashing, buckets, `equals`/`hashCode` contract, load factor, resize, treeification), `LinkedList`, `TreeMap`, `ArrayDeque`; `Comparable` vs `Comparator`; immutability (`List.of`, `Map.of`); **Spring Data JPA:** entity, repository, H2 → PostgreSQL, derived queries, transactions, N+1 awareness.

**Book:** Ch.12 Generics and Collections Framework (pp.179–200) — read §12.1–12.6 fully; **Ch.16** Serialization and RMI — **skim only** via the book TOC (serialization overview; do not invest in RMI); **Ch.17** Java Database Connectivity (pp.297+) — core JDBC sections while learning JPA (see daily split). *(Ch.13–15 are skipped; see [Book chapters skipped on purpose](#book-chapters-skipped-on-purpose).)*

**Daily split:** Mon generics ch.12 §12.1–12.3 · Tue wildcards/erasure · Wed ArrayList & HashMap internals · Thu sorting/searching, `equals`/`hashCode`, extra streams/collections practice · Fri JPA entity + repository · Sat JPA queries + **ch.17 JDBC skim** (contrast with JPA) + **ch.16 serialization skim** + review.

**Free resources:**
- Oracle Tutorials: *Generics*, *Collections*.
- dev.java: *Learn Java → Collections*, *Generics*.
- OpenJDK source (read it!): `java/util/ArrayList.java`, `java/util/HashMap.java` on https://github.com/openjdk/jdk.
- Spring Data JPA reference; Spring guide *Accessing Data with JPA* (spring.io/guides/gs/accessing-data-jpa).
- Baeldung: "Java Generics", "Guide to Java Collections", "A Guide to HashMap", "Hibernate/JPA N+1 problem".

**Hands-on exercise prompts:**
1. Write your own generic `Pair<A,B>` and a generic method `max` that works on any comparable type. Then write a method that sums a list of any `Number` subtype and explain why you chose that wildcard.
2. Implement a minimal growable array list (add, get, remove, size) and a minimal hash map (put, get) using only arrays. Describe in `notes/` how resizing works and what the time complexity of each operation is.
3. Create a class used as a `HashMap` key with a deliberately broken `hashCode` (or missing `equals`); write a test showing the failure, then fix it. Use a `record` as the key and compare.
4. Given a large list of words, find the 10 most frequent using a `Map`, and then compare timing for `ArrayList.contains` vs `HashSet.contains` on 1,000,000 elements.
5. Extend your week-3 Todo API to persist with Spring Data JPA (H2 in dev), including a derived query to find by completion status and a paging endpoint.

**Proof artifact:** `exercises/week4/` (own collections + experiments) with a benchmark table in `notes/week4.md`; the Todo API now persisting to a database with the schema and example queries documented. Tag `week4`.

---

## Week 5 (2–7 Nov) — JVM memory: stack, heap, garbage collection

**Topics:** JVM architecture (class loader, bytecode, JIT); stack frames vs heap objects; primitives vs references; pass-by-value semantics of references; object layout and `String` pool/interning; metaspace; generational heap (young: eden/survivor, old); GC concepts (reachability, mark/sweep/compact); G1 default, ZGC/Shenandoah overview; `-Xms/-Xmx/-Xss`; `OutOfMemoryError` and `StackOverflowError`; escape analysis (intro).

**Book:** none (book is silent on this). Use ch.8 §8.2 (object vs class properties, pp.103–108) only as a refresher on where instance/static state lives.

**Daily split:** Mon stack vs heap · Tue references & pass-by-value · Wed heap generations · Thu GC algorithms · Fri GC logs · Sat experiments & review.

**Free resources:**
- Oracle: *HotSpot Virtual Machine Garbage Collection Tuning Guide* (Java 21), *JVM Specification ch.2 (Run-Time Data Areas)* — https://docs.oracle.com/javase/specs/jvms/se21/html/jvms-2.html
- dev.java: *The JVM*/*Garbage Collection* learning pages; JEP 248 (G1 default), JEP 377 (ZGC production).
- Baeldung: "Stack Memory and Heap Space in Java", "Java Garbage Collection Basics", "Understanding Java GC Logs".

**Hands-on exercise prompts:**
1. Write a program that proves Java passes object references by value (a method that reassigns a parameter vs one that mutates it). Predict outputs before running.
2. Trigger a `StackOverflowError` with recursion and record the depth reached; change `-Xss` and observe. Then trigger an `OutOfMemoryError: Java heap space` deliberately with a small `-Xmx`.
3. Run an allocation-heavy loop with `-Xlog:gc*` and annotate the log: identify young vs full collections and pause times.
4. Compare `String` concatenation in a loop vs `StringBuilder` regarding objects allocated; use `==` vs `equals` and `intern()` to demonstrate the string pool.
5. Draw (by hand or tool) the stack and heap diagrams for a short program of your own with 3 objects and 2 method calls at the moment of the deepest call.

**Proof artifact:** `notes/week5-jvm-memory.md` containing annotated GC log excerpts and the two diagrams, plus the experiment programs in `exercises/week5/` with the JVM flags used documented. Tag `week5`.

---

## Week 6 (9–14 Nov) — References, leaks, profiling, JUnit testing, configuration

**Topics:** strong/soft/weak/phantom references, `WeakHashMap`, `Cleaner` (and why `finalize` is dead); common leak patterns (static collections, listeners, `ThreadLocal`, unclosed resources, caches without bounds); heap dumps and profiling basics (jcmd, jmap, VisualVM / Java Flight Recorder, Eclipse MAT); **JUnit 5** (assertions, parameterized tests, lifecycle), Mockito basics, test naming; **Spring configuration**: `application.yml`, profiles, `@ConfigurationProperties`, environment variables, secrets handling.

**Book:** ch.9 §9.6 (finalization/cleanup, pp.130–132) as a historical contrast; otherwise supplementary only.

**Daily split:** Mon reference types · Tue leak patterns · Wed heap dump & MAT · Thu JUnit 5 · Fri Mockito & test design · Sat Spring config/profiles.

**Free resources:**
- Oracle: *Troubleshooting Guide for HotSpot VM*, *Java Flight Recorder* docs; `jcmd`, `jmap` man pages.
- JUnit 5 User Guide: https://junit.org/junit5/docs/current/user-guide/ ; Mockito docs: https://site.mockito.org/
- Spring Boot reference → *Externalized Configuration*, *Profiles*, *Testing*.
- Baeldung: "Weak, Soft, and Phantom References", "Memory Leaks in Java", "Guide to JUnit 5", "Spring Boot @ConfigurationProperties".

**Hands-on exercise prompts:**
1. Write a deliberate memory leak (e.g. a static map that grows) and run it with a small heap; capture a heap dump and identify the leaking structure with a tool. Then fix it two different ways.
2. Build a small cache that uses weak references and demonstrate entries disappearing after a GC.
3. Write unit tests for your week-1 `BankAccount` and week-4 collections: at least 10 tests including edge cases and a parameterized test. Aim for tests that fail for the right reason before they pass.
4. Add tests to the Todo API: a repository test and a web-layer test (`@WebMvcTest`/`MockMvc`).
5. Move all hard-coded settings of the Todo API into typed configuration with a `dev` and `prod` profile; document how secrets are supplied via environment variables (never committed).

**Proof artifact:** Screenshot or text export of heap-dump analysis in `notes/week6-leak-hunt.md`; green test run output (`mvn test`) committed as a summary in `notes/`; configuration documented in the project README (with `.env.example` only). Tag `week6`.

---

## Week 7 (16–21 Nov) — Concurrency I: threads → `java.util.concurrent`

**Topics:** processes vs threads; `Thread`, `Runnable`; thread lifecycle; shared mutable state, race conditions; `synchronized`, `volatile`, happens-before (intro); `wait/notify` (know it, avoid it); then `java.util.concurrent`: `ExecutorService`, thread pools, `Future`, `Callable`, `ConcurrentHashMap`, atomics (`AtomicInteger`, `LongAdder`), locks (`ReentrantLock`), `CountDownLatch`, `Semaphore`, `BlockingQueue` and producer/consumer.

**Book:** Ch.11 Networking and Multithreading (pp.155–178): §11.4–11.7 (threads, synchronization, pp.165–175) are core; §11.1–11.3 (sockets) skim for the web-server example.

**Daily split:** Mon ch.11 §11.4–11.5 · Tue ch.11 §11.7 synchronization · Wed `volatile` & atomics · Thu executors · Fri concurrent collections & queues · Sat producer/consumer project.

**Free resources:**
- Oracle Tutorials: *Essential Java Classes → Concurrency*.
- dev.java: *Learn Java → Concurrency*; Java SE API docs for `java.util.concurrent`.
- Baeldung: "Guide to the Synchronized Keyword", "Java Volatile Keyword", "Guide to ExecutorService", "Guide to java.util.concurrent".
- Book reference for deep dive (optional, not free): *Java Concurrency in Practice* (Goetz).

**Hands-on exercise prompts:**
1. Write a counter incremented by many threads without synchronization; show the lost updates. Fix it three ways (`synchronized`, `AtomicInteger`, `LongAdder`) and compare correctness and timing.
2. Implement a producer–consumer pipeline using a `BlockingQueue`, with clean shutdown (no thread left hanging).
3. Process 100 URLs or simulated slow tasks with a fixed thread pool; collect results via `Future`s and report failures per task.
4. Convert a sequential word-count over multiple files into a parallel one using an executor and `ConcurrentHashMap` (merge, not manual locking).
5. Reproduce a deadlock deliberately with two locks; then fix it by lock ordering or `tryLock` with a timeout. Take a thread dump (`jstack`/`jcmd`) of the deadlocked state.

**Proof artifact:** `exercises/week7/` with the five programs, plus `notes/week7-concurrency.md` containing the thread dump of the deadlock and a table of the counter experiment timings. Tag `week7`.

---

## Week 8 (23–28 Nov) — Concurrency II: CompletableFuture, virtual threads, pitfalls + calling an LLM from Spring

**Topics:** `CompletableFuture` (`supplyAsync`, `thenApply`, `thenCompose`, `allOf`, error handling, timeouts); **virtual threads** (JEP 444) and when they help (blocking I/O) or not (CPU-bound, pinning); structured concurrency (preview — awareness); thread-safety pitfalls (check-then-act, publication, `SimpleDateFormat`, double-checked locking, unbounded pools, `parallelStream` misuse); immutability as a strategy; **Spring + LLM:** HTTP clients (`RestClient`/`WebClient`), timeouts/retries, Spring AI `ChatClient` basics, API keys via environment variables, streaming responses.

**Book:** none for modern topics; revisit ch.11 §11.7 (pp.169–175) for synchronization fundamentals.

**Daily split:** Mon `CompletableFuture` · Tue composing & error handling · Wed virtual threads · Thu pitfalls & review of week-7 code · Fri Spring HTTP client + LLM call · Sat LLM service endpoint.

**Free resources:**
- dev.java: *Virtual Threads* guide; JEP 444 (Virtual Threads), JEP 453/480 (Structured Concurrency previews).
- Oracle Java SE API docs: `CompletableFuture`, `Executors.newVirtualThreadPerTaskExecutor`.
- Spring Boot reference → *Calling REST Services* (`RestClient`); Spring AI reference → *Chat Client API*.
- Baeldung: "Guide to CompletableFuture", "Virtual Threads in Java", "Spring AI intro".
- Provider API docs of your chosen LLM (OpenAI/Anthropic/etc.).

**Hands-on exercise prompts:**
1. Call three slow simulated services in parallel with `CompletableFuture`, combine their results, and handle the case where one fails and one times out.
2. Run 10,000 tasks that each sleep for one second using (a) a fixed platform-thread pool of 200 and (b) a virtual-thread-per-task executor. Compare total time and explain.
3. Find and fix thread-safety bugs in a small piece of code you write yourself with a lazy singleton and a shared `HashMap`; prove the bug with a stress test first.
4. Add an endpoint `POST /api/ask` to a Spring Boot app that forwards a question to an LLM provider, with a timeout, graceful error mapping (provider down → 502/503), and the key read only from an environment variable.
5. Make the endpoint return a streamed response (server-sent events or chunked) and note the client-side experience.

**Proof artifact:** Spring Boot `llm-gateway` project in `exercises/week8-llm-gateway/` with a recorded `curl` session (keys redacted) in `notes/week8.md` and a short table comparing platform vs virtual thread results. Tag `week8`.

---

## Week 9 (30 Nov–5 Dec) — Project: Spring service in front of the PersonaLearn RAG endpoint (build)

**Topics:** designing a thin, robust API gateway/BFF; DTOs as records; client for the existing PersonaLearn RAG endpoint; validation (`jakarta.validation`); resilience (timeouts, retries with backoff, circuit breaker via Resilience4j); caching; logging with correlation IDs; OpenAPI docs (springdoc); simple auth (API key header) — keep secrets in env vars.

**Book:** none. Revisit ch.9 (exceptions) and ch.12 (collections) as needed.

**Daily split:** Mon requirements & API design (write the contract first) · Tue RAG client + DTOs · Wed validation & error model · Thu resilience · Fri caching & logging · Sat integration test with a fake RAG server.

**Free resources:**
- Spring Boot reference → *Actuator*, *Validation*, *Caching*; Spring guides *Building a RESTful Web Service*, *Validating Form Input*.
- Resilience4j docs: https://resilience4j.readme.io/
- springdoc-openapi docs: https://springdoc.org/
- Baeldung: "Spring Boot Validation", "Guide to Resilience4j", "Spring Boot Actuator".

**Hands-on exercise prompts:**
1. Write a one-page API contract (endpoints, request/response JSON, error shapes, status codes) for the service *before* writing code.
2. Implement a client class for the PersonaLearn RAG endpoint behind an interface so it can be faked in tests; handle slow and failing upstreams explicitly.
3. Add request validation (length limits, required fields) and a uniform JSON error response.
4. Add retry with backoff and a circuit breaker; show in logs what happens when the upstream is down.
5. Add a bounded in-memory cache for repeated questions with an expiry; measure hit rate.

**Proof artifact:** Service running locally in `exercises/week9-rag-gateway/` with the API contract in its README and an integration test using a fake upstream, all green. Tag `week9`.

---

## Week 10 (7–12 Dec) — Project: harden, test, document, demo

**Topics:** observability (Actuator, metrics, structured logs); load and soak testing (simple), memory/GC check of your own service using week 5–6 tools; concurrency review (virtual threads enabled via `spring.threads.virtual.enabled`, thread-safety audit); containerization (Dockerfile, layered jars); CI basics (GitHub Actions: build + test); documentation and demo.

**Book:** none.

**Free resources:**
- Spring Boot reference → *Container Images*, *Actuator Metrics*, *Virtual threads* section.
- GitHub Actions docs: https://docs.github.com/actions ; Docker docs: https://docs.docker.com/
- Baeldung: "Dockerizing a Spring Boot Application", "Spring Boot Virtual Threads".

**Hands-on exercise prompts:**
1. Run a simple load test against the gateway (any free tool), then capture GC logs and a heap histogram under load; record findings.
2. Turn virtual threads on and off and compare throughput/latency under a slow upstream; decide which to ship and justify.
3. Write a Dockerfile for the service and run it with configuration only via environment variables.
4. Add a GitHub Actions workflow that builds and runs tests on every push (no secrets in the workflow file).
5. Write the README: architecture sketch, how to run, config table, known limitations, and what you would do next.

**Proof artifact:** A 3–5 minute screen-recorded (or written) demo of the working service plus a green CI run link; `notes/retrospective.md` listing what you can now do, what is weak, and the next 3 learning goals. Tag `week10`.

---

## Buffer week (14–19 Dec) — catch-up and polish

**Use for:** finishing unfinished exercises; revisiting topics flagged "still fuzzy" in weekly notes; refactoring early exercises with what you now know; re-doing 3 self-check sets cold; repo tidy-up (README accuracy, no stray secrets: run `git log -p | grep -i -E "key|secret|token"` yourself).

**Hands-on prompts:**
1. Pick your weakest topic from the weekly notes and re-implement its main exercise from a blank file, without looking at your old code.
2. Refactor one early exercise using records, streams or interfaces where they genuinely improve it, and justify each change.
3. Read one real open-source class (e.g. `ArrayList`, a Spring `@Configuration`) and write a one-page explanation of how it works.

**Proof artifact:** Updated `notes/retrospective.md` with a revised roadmap for January (e.g. Spring Security, messaging, Spring AI RAG internals, JVM tuning), and a clean repo (all tests green).

---

## Weekly rhythm reminders

- **Sat:** mini-review: re-attempt one exercise you found hard; update `notes/`.
- **Sunday:** rest (or optional reading only).
- If you miss a day, do **not** double up — drop the lowest-value exercise and continue.
- If stuck > 15 min: write the exact error/question in `notes/` and request a review (agents may give hints and review feedback, **not** solutions).
