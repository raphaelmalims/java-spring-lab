# Lecture 13 — Spring Boot “hello REST”: `@RestController`, JSON records, and HTTP status

**Date:** Fri 23 Oct 2026 · **Book:** *Object-Oriented Programming and Java*, 2nd ed. — **no new book chapter today** (Week 3 Fri = **Spring Boot** per `CURRICULUM.md`; ch.13–15 GUI/applets/servlets remain **skipped** — the web path is Spring Boot, not hand-written servlets).

**Time plan (1 h):** 10 min recap (Thu `java.nio.file` + `Scanner`) · 20 min read this lecture + [Building a RESTful Web Service](https://spring.io/guides/gs/rest-service/) · 30 min hands-on (see §13 — curriculum **#4** only; problem statement in `CURRICULUM.md` only).

**Goal for today:** generate a **Spring Boot** project (e.g. [start.spring.io](https://start.spring.io/)), run it locally, expose **`GET`** and **`POST`** endpoints under a REST-style path, return **JSON** using **records** as DTOs, and choose **HTTP status codes** (including **400** with a **JSON error body** for bad input) — without servlet programming from the book.

> **Pacing note:** Week 3 in `CURRICULUM.md` schedules **Mon = ch.9 §9.1–9.5**, **Tue = §9.6 + try-with-resources**, **Wed = ch.10 streams & files**, **Thu = `java.nio.file` + `Scanner`**, **Fri = Spring Boot "hello REST"** (this file), **Sat = REST + error handling**. Do not invent schedule changes inside this lecture.

---

## 1. Recap hook (Thu 22 Oct — use the first 10 minutes)

Before Spring Boot, from memory in `notes/`:

1. **Three bullets from yesterday:** e.g. **`Path.of`** vs string concatenation; **`Files.copy`** vs byte-stream copy for **#3**; why **`Files.lines`** needs try-with-resources; **`Scanner.nextLine()`** + parse vs **`nextInt()`** for **#1** line numbers.
2. **One self-check from Lecture 12:** e.g. when **`IOException`** at file open is a different class of problem than a malformed line in **#1**; which charset you standardized on for Week 3 text files.
3. **Skim** **#1** / **#3** progress — note one place where “report bad input and continue” on Thu maps to “return **400** and a clear JSON message” on Fri (Sat adds centralized handlers for **#5**).

Thu was **files and parsing on disk**. Fri is the same **discipline** over **HTTP**: valid vs invalid input, correct status, structured response — still **no solution code** from agents; you type the app yourself.

*(Book ch.13–15 — GUI, applets, servlets — remain **skipped**; see `CURRICULUM.md` [Book chapters skipped on purpose](#book-chapters-skipped-on-purpose).)*

---

## 2. Why Spring Boot instead of book servlets (ch.15 skipped)

The book’s servlet chapter describes **container-managed** `HttpServlet` classes, **`doGet`/`doPost`**, and deployment descriptors — a valid mental model for “request in, response out”, but not how most **new** Java services are built.

| Book path (skipped) | This lab’s path (Week 3+) |
|---|---|
| Write servlets + wire in a container by hand | **Spring Boot** embeds a servlet stack and auto-configures it |
| Manual JSON serialization | **Jackson** on the classpath maps **records** / beans ↔ JSON |
| Ad-hoc status codes in `HttpServletResponse` | **`ResponseEntity`** (and annotations) express status + body explicitly |

You still need the **HTTP ideas**: methods (**GET**, **POST**), paths, status codes (**200**, **201**, **400**), and a **JSON** body. Spring Boot is the **framework** that applies those ideas with less boilerplate than raw servlets.

---

## 3. Create the project — start.spring.io

Use the [Spring Initializr](https://start.spring.io/) (same engine as the site) with settings aligned to the lab:

| Setting | Suggestion |
|---|---|
| **Project** | Maven (Gradle is fine if you already prefer it) |
| **Language** | Java |
| **Spring Boot** | Current stable 3.x line |
| **Java** | **21** (matches `CURRICULUM.md` JDK target) |
| **Packaging** | Jar |
| **Dependencies** | **Spring Web** (brings embedded Tomcat + Jackson for JSON) |

Download, unzip, open in IntelliJ IDEA Community or VS Code + Java extensions. First sanity check:

```bash
./mvnw spring-boot:run
```

Default port **8080**; hitting the root may 404 until you add controllers — that is expected.

**Project home for Week 3 proof:** develop toward **`exercises/week3-todo-api/`** per curriculum **Proof artifact** (running app + `curl` transcript in `notes/week3.md` later in the week). Scratch folders under `exercises/2026-10-23-spring-hello-rest/` are fine until you consolidate.

**Secrets:** never commit API keys or `.env` — curriculum ground rule.

---

## 4. Application entry and component scan

A typical main class:

```java
@SpringBootApplication
public class TodoApiApplication {
    public static void main(String[] args) {
        SpringApplication.run(TodoApiApplication.class, args);
    }
}
```

**`@SpringBootApplication`** combines configuration that **component-scans** your package (and subpackages) for **`@RestController`**, **`@Service`**, etc. Put controllers in a subpackage (e.g. `...web` or `...api`) so they are discovered.

**Run configuration:** same **`main`** as any Java program; Maven **`spring-boot:run`** is equivalent for CLI workflows.

---

## 5. `@RestController` — JSON over HTTP

**`@RestController`** = **`@Controller`** + **`@ResponseBody`** on each handler method: return values become the **response body** (not a view name).

Minimal “hello” shape (illustration only — your todo API follows curriculum **#4**):

```java
@RestController
public class HelloController {

    @GetMapping("/hello")
    public String hello() {
        return "Hello";
    }
}
```

For APIs, prefer **structured JSON** (records) over plain **`String`** bodies.

**Contrast with Thu:** instead of **`System.out`** / a file report for bad input, the client sees an **HTTP status** and **JSON** payload.

---

## 6. Request mapping — paths and methods

| Annotation | HTTP method | Typical use |
|---|---|---|
| **`@GetMapping("/path")`** | GET | Read / list resources |
| **`@PostMapping("/path")`** | POST | Create a resource |
| **`@PutMapping`**, **`@DeleteMapping`** | PUT, DELETE | Update / delete (later weeks) |

Curriculum **#4** asks for:

- **`GET /api/todos`** — list todos stored **in memory**
- **`POST /api/todos`** — add a todo from the request body

Use a **class-level** prefix if you like consistency:

```java
@RestController
@RequestMapping("/api/todos")
public class TodoController {
    // GET on "" or "/" vs POST on "" — design paths so they match #4 literally
}
```

**Path variables** (**`GET /api/todos/{id}`**) are curriculum **#5** (**Sat**), not required to finish **#4** today — but reading the mapping table now avoids rework tomorrow.

**Query parameters** (`?completed=true`) are optional for **#4**; derived queries come in Week 4 JPA.

---

## 7. JSON via records — request and response DTOs

**Records** (Week 1–2) fit **immutable API shapes**: JSON maps cleanly to record components when names match (or you add Jackson annotations later if needed).

| Role | Record idea | JSON direction |
|---|---|---|
| **Response** | e.g. todo with `id`, `title`, `completed` | Server → client |
| **Request body** | e.g. create payload with `title` (no client-supplied `id` yet) | Client → server via **`@RequestBody`** |
| **Error body** | e.g. `message`, optional `field` | Server → client on **400** / later **404** |

Example **response** record (field names are yours; match what you document in `notes/`):

```java
public record TodoResponse(long id, String title, boolean completed) {}
```

**POST** handler sketch (you implement storage and validation):

```java
@PostMapping
public ResponseEntity<?> create(@RequestBody TodoCreateRequest request) {
    // validate → 201 + body, or 400 + JSON error (curriculum #4)
}
```

**Jackson** deserializes JSON to records when Spring Web is on the classpath. Invalid JSON often yields **400** before your method runs — distinguish that from **your** domain validation (empty title, etc.) in **`notes/`** when you test with **`curl`**.

**Week 2 decision guide:** todo **response** / **request** DTOs → **record**; **in-memory store** with mutable list + id generation → **class** or careful service design (identity and lifecycle).

---

## 8. `ResponseEntity` — status + body together

Returning a bare record from a **`@GetMapping`** usually yields **200 OK**. Creation and errors need explicit status:

| Status | When (curriculum **#4** focus) |
|---|---|
| **200 OK** | **GET** list succeeds (even if empty list) |
| **201 Created** | **POST** created a new todo; body often includes the created resource |
| **400 Bad Request** | Client sent unusable input — curriculum requires a **JSON error body**, not an empty body |

**`ResponseEntity`** carries status, optional headers, and body:

```java
return ResponseEntity.status(HttpStatus.CREATED).body(createdTodo);
return ResponseEntity.badRequest().body(new ErrorBody("title must not be blank"));
```

Alternative: **`@ResponseStatus(HttpStatus.CREATED)`** on the method when you always return **201** with a body — **`ResponseEntity`** is clearer when one method returns **201** *or* **400**.

**Location header** (optional stretch): **`ResponseEntity.created(uri).body(...)`** — not required by **#4**; useful habit for REST APIs.

---

## 9. In-memory storage (curriculum **#4** — design, not solution)

**#4** requires **in-memory** storage (no database until Week 4 **#5** extension).

Design choices **you** make:

| Concern | Options to decide |
|---|---|
| **Where state lives** | Field on a **`@RestController`** (small apps) vs **`@Service`** bean (better testability) |
| **Id generation** | Monotonic **`long`**, **`UUID`**, etc. |
| **Thread safety** | Single-threaded dev testing vs **`ConcurrentHashMap`** / synchronized list if you stress-test |
| **List vs map** | List for order; map by id speeds **#5** lookup |

Document in **`notes/`** why you picked one approach. **Sat** refactors error handling without changing the core **GET/POST** contract if you keep paths stable.

---

## 10. Bad input → 400 + JSON error body (today’s bar)

Curriculum **#4** explicitly: **return correct HTTP status codes and a JSON error body for bad input**.

| Thu / file (**#1**) | Fri / HTTP (**#4**) |
|---|---|
| Malformed line → message + line number, continue | Invalid create payload → **400**, JSON explains what failed |
| Missing file → **`IOException`** policy | Missing required JSON field → **400** (your validation) |
| Sum of valid numbers on stdout | **GET** returns JSON array of todos |

**Today:** handle validation **inside** the controller or a small helper — return **`ResponseEntity.badRequest().body(...)`** with a **record** (or type) serialized as JSON.

**Sat (curriculum daily split):** **REST + error handling** — **`@ExceptionHandler`** / **`@RestControllerAdvice`** for **#5** (**404** with structured error) and to **centralize** the **400** pattern you start today. Same arc as “catch at boundary vs per-line” on **#1**.

Do **not** require **`@RestControllerAdvice`** to call **#4** done — but read the Spring guide section on error handling so Sat is not your first exposure to the idea.

---

## 11. Testing with curl (habit for `notes/week3.md`)

Examples use placeholder host/port; adjust paths to match your app:

```bash
curl -s http://localhost:8080/api/todos
curl -s -X POST http://localhost:8080/api/todos \
  -H 'Content-Type: application/json' \
  -d '{"title":"Learn Spring"}'
curl -s -X POST http://localhost:8080/api/todos \
  -H 'Content-Type: application/json' \
  -d '{"title":""}'
```

Save transcripts showing **200**, **201**, and **400** with JSON bodies when you reach the Week 3 **proof artifact** (curriculum also lists **404** after **#5** on Sat).

---

## 12. Free resources (Week 3 — links only)

**Fri focus:** Spring Boot web + JSON. Full Week 3 list lives in `CURRICULUM.md`; use these today:

- [Spring Guide — Building a RESTful Web Service](https://spring.io/guides/gs/rest-service/) (**primary read today**)
- [Spring Boot reference — Web](https://docs.spring.io/spring-boot/docs/current/reference/html/web.html) (Servlet stack overview)
- [Spring Boot reference — Error handling](https://docs.spring.io/spring-boot/docs/current/reference/html/web.html#web.servlet.spring-mvc.error-handling) (skim for Sat)
- [Java SE 21 API — records](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/Record.html) (DTO recap)
- Baeldung: [Spring Boot REST Controller](https://www.baeldung.com/spring-controller-vs-restcontroller), [ResponseEntity](https://www.baeldung.com/spring-response-entity)

*(Oracle I/O tutorials were Thu; stay on Spring for Fri.)*

---

## 13. Glossary

**Spring Boot** · **Spring Initializr** / **start.spring.io** · **`@SpringBootApplication`** · **`@RestController`** · **`@RequestMapping`** · **`@GetMapping`** · **`@PostMapping`** · **`@RequestBody`** · **DTO** · **record** · **JSON** · **Jackson** · **`ResponseEntity`** · **`HttpStatus`** · **201 Created** · **400 Bad Request** · embedded **Tomcat** · **`spring-boot:run`** · **`Content-Type: application/json`** · in-memory store · **`curl`**

---

## 14. Common pitfalls (Fri)

1. **Missing `spring-boot-starter-web`** — app runs but no REST stack / Jackson as expected.
2. **Wrong package** — controller not under the main class package tree → **404** on all routes.
3. **POST without `@RequestBody`** — empty or null fields; always send **`Content-Type: application/json`** from clients.
4. **400 with plain text or empty body** — curriculum **#4** asks for **JSON error body**.
5. **201 vs 200 on POST** — creation should be **201** when a resource was added (curriculum proof set expects **201** in the transcript).
6. **Storing todos in static fields** without noticing test/restart behavior — acceptable for **#4**; document that restart clears memory.
7. **Jumping ahead to JPA** — Week 4 **#5** extends this API with persistence; finish in-memory **#4** first.
8. **Reading book ch.15 servlets** instead of the Spring guide — time-box Fri on the modern path per curriculum.

---

## 15. Self-check (answer in `notes/`, then check Spring guide / docs)

1. What dependency does **Spring Web** add, and why do you need it for JSON APIs?
2. How is **`@RestController`** different from **`@Controller`** for a JSON API?
3. Which HTTP status should **POST /api/todos** return when a todo is successfully created?
4. Why is a **record** a reasonable type for a todo returned as JSON?
5. When should you use **`ResponseEntity`** instead of returning a record directly?
6. How does “bad line in **#1**” correspond to “bad JSON or validation in **#4**”?

*Stuck for more than 15 min? Write the exact error/question in `notes/` and ask for a review — hints and feedback, not solutions.*

---

## 16. Hands-on (Fri — curriculum prompt only; you write all code)

Week 3 proof set: **five prompts** in `CURRICULUM.md` Week 3 (**#1–#5**). Problem statements live in the curriculum only — **no solution code** in this lecture.

**Fri 23 Oct (Spring Boot "hello REST"):** today’s primary hands-on is curriculum **#4** in **`exercises/week3-todo-api/`** (or scratch until you consolidate there).

### Curriculum **#4** (problem statement only)

Build a Spring Boot app with **`GET /api/todos`** and **`POST /api/todos`** storing items in memory; return correct HTTP status codes and a JSON error body for bad input.

| Other prompts | Fri guidance |
|---|---|
| **#1–#3** | File I/O week — continue in parallel only if incomplete; do not skip **#4** today |
| **#5** | **Sat** — `GET /api/todos/{id}`, **404**, centralized exception handling |

Work toward the curriculum **Proof artifact**: running Spring Boot app in **`exercises/week3-todo-api/`** with a **`curl` transcript** in **`notes/week3.md`** showing **201**, **200**, **400**, and (after Sat) **404**; plus the file-parsing exercise from **#1**. Tag **`week3`** when the week is complete.

Closing habit (2 min): one line in **`notes/`** — what clicked / what confused you about **`@RestController`**, records, and **`ResponseEntity`**.

---

## 17. Bridge to Sat (per `CURRICULUM.md` Week 3)

**Tomorrow:** **REST + error handling** — add **`GET /api/todos/{id}`** (curriculum **#5**), return **404** with a structured error when missing, and use **`@ExceptionHandler`** / **`@RestControllerAdvice`** so **400** / **404** bodies are consistent (Spring Boot reference → error handling; Baeldung **@ControllerAdvice** per curriculum free resources).

| Fri (**#4**) | Sat (**#5** + error handling) |
|---|---|
| **GET** list, **POST** create | **GET** by id |
| **400** JSON for bad input (local handling OK) | **404** JSON for missing id |
| In-memory store | Same store; centralize exception mapping |

Week 4 **Fri** then moves the todo API to **JPA** (**#5** extension in Week 4) — keep URLs and JSON shapes stable where you can.

Do not jump to Swing (ch.13) or servlets (ch.15); finish Week 3 REST proof on the Spring path.
