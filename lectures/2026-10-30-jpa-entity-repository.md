# Lecture 19 — Spring Data JPA: entity, repository, and H2 in dev

**Date:** Fri 30 Oct 2026 · **Book:** *Object-Oriented Programming and Java*, 2nd ed. — **no new book chapter today** (Week 4 Fri = **JPA entity + repository** per `CURRICULUM.md`; ch.13–15 GUI/applets/servlets remain **skipped**). **Ch.12** (Generics and Collections Framework, pp.179–200) gave you **`List`**, **`Map`**, and framework **interfaces** — persistence today replaces “todos in a **`HashMap`**” with **rows in a table** mapped to **entity** objects; that is context only, not a reread assignment.

**Time plan (1 h):** 10 min recap (Thu **`equals`/`hashCode`**, **#3–#4**) · 20 min read this lecture + [Accessing Data with JPA](https://spring.io/guides/gs/accessing-data-jpa/) (through entity + repository; stop before query-method deep dives) · 30 min hands-on (curriculum **#5** **start** — entity + repository, first persisted todos; problem statement in `CURRICULUM.md` only).

**Goal for today:** add **Spring Data JPA** and an **H2** database to your Week 3 todo API shape: define a **`@Entity`**, a **`JpaRepository`** (or **`CrudRepository`**) interface, configure **H2 for dev**, and wire **GET/POST** (and existing **GET by id** if present) so todos **survive an app restart** — without hand-written SQL, JDBC **`Connection`** code, derived query methods, paging, or production PostgreSQL (those belong to **Sat** and later weeks per curriculum).

> **Pacing note:** Week 4 in `CURRICULUM.md` schedules **Mon = ch.12 §12.1–12.3**, **Tue = wildcards/erasure**, **Wed = ArrayList & HashMap internals**, **Thu = sorting, `equals`/`hashCode`, collections practice** (`lectures/2026-10-29-equals-hashcode-collections-practice.md`), **Fri = JPA entity + repository** (*this file*), **Sat = JPA queries + ch.17 JDBC skim + ch.16 serialization skim + review**. Do not invent schedule changes inside this lecture.

---

## 1. Recap hook (Thu 29 Oct — use the first 10 minutes)

Before persistence, from memory in `notes/`:

1. **Three bullets from Lecture 18:** e.g. **`equals`/`hashCode` contract** for **`HashMap`** keys; **`Comparable`** vs **`Comparator`**; why **`ArrayList.contains`** is **O(n)** vs **`HashSet`** average **O(1)**.
2. **One self-check from Thu:** e.g. what breaks when **`equals`** is overridden but **`hashCode`** is not; why **`record`** keys are attractive for **immutable** value keys in memory.
3. **Skim Week 3 todo API:** confirm **`GET /api/todos`**, **`POST /api/todos`**, and **`GET /api/todos/{id}`** with **201/200/400/404** still match your **`notes/week3.md`** contract — **#5** **extends** that app; it does not replace HTTP semantics.

Thu was **in-memory** structures and key correctness. Fri is the same **todo domain** with **durability**: identity moves from “next id in a map” to **primary key in a table**, while your **JSON records** for API responses can stay as they were.

*(Book ch.13–15 remain **skipped**; see `CURRICULUM.md` [Book chapters skipped on purpose](#book-chapters-skipped-on-purpose).)*

---

## 2. Week 4 arc (where Fri sits)

| Day | Theme (curriculum) | Lecture file | Carry into Fri |
|---|---|---|---|
| **Mon 26** | generics ch.12 §12.1–12.3 | `2026-10-26-ch12-generics.md` | typed collections |
| **Tue 27** | wildcards / erasure | `2026-10-27-ch12-wildcards-erasure.md` | **`List<Todo>`** in services |
| **Wed 28** | ArrayList & HashMap internals | `2026-10-28-arraylist-hashmap-internals.md` | in-memory map mental model |
| **Thu 29** | sorting, equals/hashCode, practice | `2026-10-29-equals-hashcode-collections-practice.md` | entity **`equals`/`hashCode`** caution |
| **Fri 30** | JPA entity + repository | *this file* | **#5** start (persist todos) |
| **Sat 31** | JPA queries + JDBC/serialization skim + review | *(Sat lecture when published)* | derived queries, paging, proof **`week4`** |

**Book this week (already assigned Mon–Tue):** Ch.12 **§12.1–12.6**. Fri does **not** assign ch.16 or ch.17 — **Sat** skims serialization and JDBC **after** you have a working entity layer.

**Modern thread (Week 4 Topics — today’s slice only):** **Spring Data JPA** — **entity**, **repository**, **H2 → PostgreSQL** (PostgreSQL is **not** today’s hands-on; H2 in dev is).

---

## 3. From in-memory map to relational rows (conceptual)

Week 3 stored todos in a **`Map<Long, Todo>`** or list on the heap — fast, simple, **gone when the process exits**.

| In-memory (Week 3) | JPA + database (Week 4 **#5**) |
|---|---|
| **`put(id, todo)`** | **`repository.save(entity)`** → **INSERT** or **UPDATE** |
| **`get(id)`** | **`repository.findById(id)`** |
| **`values()`** for list | **`repository.findAll()`** (fine for tiny dev datasets) |
| You owned id generation | Often **`@GeneratedValue`** on **`@Id`** |

**JPA** (Jakarta Persistence API) defines **annotations** and **`EntityManager`** semantics; **Hibernate** is the common implementation on the classpath. **Spring Data JPA** adds **repository interfaces** so you declare **`findById`**, **`save`**, **`delete`** without boilerplate implementations.

You are **not** writing **`DriverManager.getConnection`** today — that is what **Sat’s ch.17 JDBC skim** contrasts with JPA. Today: **object ↔ table mapping** and **one repository** wired into your existing controller/service.

Official overview: [Spring Data JPA reference — Introduction](https://docs.spring.io/spring-data/jpa/reference/jpa.html), [Spring Boot — Data Access](https://docs.spring.io/spring-boot/reference/data/index.html).

---

## 4. Dependencies — extend your todo project

Add to an existing Maven **`pom.xml`** (or regenerate via [start.spring.io](https://start.spring.io/) into the same app tree):

| Dependency | Role |
|---|---|
| **Spring Data JPA** | Repositories, transaction integration |
| **H2 Database** | Embedded JDBC database for **dev** (in-memory or file URL) |
| **Spring Web** | Already on Week 3 app |

Optional for local inspection: **H2 Console** via **`application.properties`** / **`application.yml`** (Spring Boot docs — enable only in **dev**, never expose in prod without understanding the risk).

**JDK 21** and **Spring Boot 3.x** use **`jakarta.persistence.*`** (not legacy **`javax.persistence`**).

**Secrets:** curriculum ground rule — no committed **`.env`**, no production DB passwords in git.

---

## 5. The JPA entity — mapping a class to a table

An **entity** is a **mutable class** (not a **`record`**) representing one row — aligns with Week 2 decision guide: **class** when there is **lifecycle** and **identity**.

Typical building blocks (names and fields are **yours**; match your todo domain):

| Annotation / element | Purpose |
|---|---|
| **`@Entity`** | Marks class as managed persistence type |
| **`@Table(name = "...")`** | Optional table name (default often simple class name) |
| **`@Id`** | Primary key field |
| **`@GeneratedValue`** | Strategy for DB-generated ids (**`IDENTITY`**, **`SEQUENCE`**, etc.) |
| **Column mapping** | Default column names from field names; **`@Column`** when you need nullability, length, unique |

**Example shape (illustration only — you implement in **#5**):**

```java
@Entity
public class TodoEntity {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String title;
    private boolean completed;

    protected TodoEntity() { } // JPA requires no-arg constructor

    // constructors, getters/setters or accessors you choose
}
```

**API JSON vs entity:**

| Layer | Type | Why |
|---|---|---|
| HTTP request/response | **`record`** (Week 3) | Immutable DTO; stable JSON contract |
| Database row | **`@Entity` class** | JPA needs mutable state, proxies, lazy loading hooks later |

Map **entity → response record** in a service or controller method (small private mapper methods are enough today).

**`equals`/`hashCode` on entities (Thu link):** do **not** blindly include all fields like a **`record`**. Common practice: **`equals`/`hashCode` based on `id`** once assigned, or a **stable business key** — avoid collections keyed by entities with naive “all fields” equality. Fri hands-on can defer fancy equality until **Sat** review if you only use **`findById`**.

---

## 6. Spring Data repository — interface, no implementation class

Create a **Spring Data JPA** repository interface in a package under your application root (component scan):

```java
public interface TodoRepository extends JpaRepository<TodoEntity, Long> {
}
```

**`JpaRepository<TodoEntity, Long>`** gives **`save`**, **`findById`**, **`findAll`**, **`deleteById`**, **`count`**, etc. Spring generates the implementation at runtime.

| Method (inherited) | Maps to your API habit |
|---|---|
| **`save(entity)`** | **POST** create (and later update) |
| **`findById(id)`** | **GET /{id}** → **`Optional`** → **404** if empty |
| **`findAll()`** | **GET** list |

Inject the repository into your **`@RestController`** or (preferred) **`@Service`** with constructor injection:

```java
public TodoService(TodoRepository repository) {
    this.repository = repository;
}
```

Remove or bypass the old **in-memory map** for todo storage once persistence works — keep the **same URL paths** and status codes from Week 3.

**Do not** add **`findByCompleted`** or **`Pageable`** methods today — curriculum **#5** lists **derived query** and **paging** for **Sat**.

Guide walkthrough: [Accessing Data with JPA](https://spring.io/guides/gs/accessing-data-jpa/) — follow through **Creating repository interfaces** and running the app; skip ahead sections on **Query methods** until tomorrow.

---

## 7. H2 configuration for development

In **`application.properties`** or **`application.yml`** (dev profile if you already use profiles):

| Property idea | Typical dev value | Notes |
|---|---|---|
| **JDBC URL** | `jdbc:h2:mem:todos` or file-based `jdbc:h2:file:./data/todos` | **mem** resets on restart unless you use file URL |
| **Driver** | auto-detected with H2 on classpath | |
| **Username / password** | often `sa` / empty in dev | not secrets for local H2 |
| **DDL** | `spring.jpa.hibernate.ddl-auto=update` (dev only) | creates/updates schema from entities — learn first, migrate properly in real projects later |
| **Show SQL** (optional) | `spring.jpa.show-sql=true` | helps you see INSERT/SELECT while learning |

**H2 vs PostgreSQL (curriculum Topics):** production-style **PostgreSQL** is a **later** step when you harden the app; Week 4 proof expects **H2 in dev** documented in **`notes/week4.md`**.

After first run, verify: create a todo via **POST**, restart the app — with **file** H2 or **`ddl-auto`** + same DB file, todos should still list on **GET**; with **pure mem** without persistence URL, understand why data disappears (document which you chose).

---

## 8. Wiring the REST layer — same HTTP contract, new store

Preserve Week 3 behaviour:

| Endpoint | Persistence change |
|---|---|
| **GET /api/todos** | **`repository.findAll()`** → map to response records |
| **POST /api/todos** | Build entity from request DTO → **`save`** → **201** + body with generated **id** |
| **GET /api/todos/{id}** | **`findById`** → **200** or **404** + JSON error ( **`@RestControllerAdvice`** from Week 3 Sat still applies) |
| **400** on bad input | Unchanged — validation before **`save`** |

**Transactions (awareness only):** Spring Data JPA **`save`** runs in a transaction by default on service boundaries. You do **not** need **`@Transactional`** on every controller method today; if you extract a **`@Service`**, putting **`@Transactional`** on the service class is a common pattern — **Sat** and Week 6 testing go deeper.

**N+1 queries:** curriculum **Topics** mention **N+1 awareness** — not today’s exercise. If **`show-sql`** prints one query per todo when listing, note it in **`notes/`** for **Sat**; fixing it belongs with **query design**, not Fri’s entity bootstrap.

---

## 9. Book ch.12 ↔ JPA (brief paraphrase, not a reread)

Ch.12 introduced **collection interfaces** (**`List`**, **`Map`**, **`Set`**) and **generics** so structures are **type-safe**. A **`JpaRepository`** is another ** abstraction over storage**: instead of **`HashMap<Long, Todo>`**, the “container” is the **database**, and the **entity** is the element type. **Wildcards and erasure** (Tue) still matter when you return **`List<TodoResponse>`** from **`findAll()`** streams mapping.

Ch.12 did **not** cover SQL or ORM — that gap is filled by **Spring** docs and **Sat’s JDBC skim**, not by ch.13–15.

---

## 10. Design ties to your stack

| Layer | Fri concepts |
|---|---|
| **Week 3 REST** | Same paths and status codes; swap store |
| **Week 2 guide** | DTO **`record`** at edge; **`@Entity` class** at persistence |
| **Thu contract** | Careful **`equals`/`hashCode`** if entities ever land in **`Set`** or cache keys |
| **Week 4 proof (`week4`)** | Todo API persisting + schema notes in **`notes/week4.md`** — complete **#5** through **Sat** |

---

## 11. Free resources (Week 4 Fri — links only)

Full Week 4 list lives in **`CURRICULUM.md`**; use these today:

- [Spring Guide — Accessing Data with JPA](https://spring.io/guides/gs/accessing-data-jpa/)
- [Spring Data JPA reference](https://docs.spring.io/spring-data/jpa/reference/)
- [Spring Boot — SQL databases](https://docs.spring.io/spring-boot/reference/data/sql.html)
- [H2 Database](https://www.h2database.com/html/main.html) (embedded mode docs)
- Baeldung (conceptual): [Introduction to Spring Data JPA](https://www.baeldung.com/the-persistence-layer-with-spring-data-jpa) — read entity/repository sections; defer advanced query articles to **Sat**

Oracle/dev.java **Collections** links from Mon–Thu remain relevant for **in-memory tests** and **#1–#4** proof code, not for replacing JPA docs today.

---

## 12. Glossary

**ORM** · **JPA / Jakarta Persistence** · **Hibernate** · **`@Entity`** · **`@Id`** · **`@GeneratedValue`** · **`JpaRepository`** · **`save` / `findById` / `findAll`** · **H2 (dev)** · **`ddl-auto`** · **DTO record vs entity class** · **primary key** · **`Optional`** · **persistence context** (name only)

---

## 13. Common pitfalls (Fri)

1. **Using a `record` as `@Entity`** — JPA expects a no-arg constructor and mutable fields; keep **records** at the API boundary.
2. **Forgetting `@Entity` or `@Id`** — startup errors or tables without primary keys.
3. **Client sends `id` on POST** — ignore or reject; let **`@GeneratedValue`** assign ids.
4. **Returning `Optional` directly as JSON** — unwrap **`findById`**; map to **404** or response DTO.
5. **Mixing in-memory and JPA stores** — confusing duplicates; remove the old map once migration works.
6. **Committing H2 console open in prod config** — dev-only habit.
7. **Expecting derived queries today** — **`findByCompleted`**, **`Pageable`** are **Sat** per curriculum **#5** full prompt.
8. **Book detour ch.13–15 or deep JDBC today** — stay on entity + repository path.

---

## 14. Self-check (answer in `notes/`, then start **#5**)

1. What problem does **`JpaRepository.save`** solve that a **`HashMap.put`** did not?
2. Why keep **JSON DTOs** as **records** while the **entity** is a **class**?
3. What does **`spring.jpa.hibernate.ddl-auto=update`** do in dev (and why is it not a migration strategy for production)?
4. After **`findById`**, how do you preserve Week 3 **404** behaviour?
5. Where will **derived queries** and **paging** enter the plan (**which curriculum day**)?
6. Name one Thu **`equals`/`hashCode`** rule that still matters if two **`TodoEntity`** instances with the same **id** are in a **`HashSet`**.

*Stuck for more than 15 min? Write the exact error/question in `notes/` and ask for a review — hints and feedback, not solutions.*

---

## 15. Hands-on (Fri — curriculum **#5** start only; you write all code)

Week 4 proof set: **five prompts** in **`CURRICULUM.md` Week 4 (**#1–#5**). Problem statements live in the curriculum only — **no solution code** in this lecture.

**Fri 30 Oct (after this read):** **start** curriculum **#5** — entity + repository + H2; todos persist across restart (with the H2 mode you document). **Do not** require derived queries, paging endpoint, or PostgreSQL switch today — those are **#5** completion items on **Sat** per daily split.

### Curriculum **#5** (Fri scope — problem statement paraphrase)

From **`CURRICULUM.md` Week 4 hands-on **#5**:** extend your week-3 Todo API to persist with Spring Data JPA (**H2 in dev**), including (in the **full** week prompt) a derived query to find by completion status and a paging endpoint.

| Part | Fri (today) | Sat (not today) |
|---|---|---|
| **Entity + table mapping** | **`@Entity`**, fields match todo domain | refine schema notes for proof |
| **Repository** | **`JpaRepository`**, **`save`**, **`findById`**, **`findAll`** | **derived query** by completion status |
| **H2 dev config** | JDBC URL, **`ddl-auto`**, optional **`show-sql`** | contrast with **ch.17 JDBC** skim |
| **REST wiring** | **GET/POST** (+ **GET by id**) use repository | **paging** endpoint |
| **Documentation** | In **`notes/`**: H2 URL choice, table/columns you see | example queries + benchmark table from **#1–#4** |

Work in your existing **`exercises/week3-todo-api/`** tree (or consolidate toward **`exercises/week4/`** when ready). Finish **#1–#4** before **`week4`** tag if still open — Fri primary is **#5** bootstrap.

Closing habit (2 min): one line in **`notes/`** — what clicked / what confused you about **entity vs DTO** or **generated ids**.

---

## 16. Bridge to Sat (per `CURRICULUM.md` Week 4)

**Tomorrow (Sat):** finish **#5** — **derived queries** (e.g. find by **completion status**), a **paging** endpoint, **JPA queries** practice, plus **book skims**: **ch.17 JDBC** (drivers, **`Connection`**, prepared statements — what JPA abstracts) and **ch.16 serialization** (**`Serializable`** overview; treat RMI as legacy; prefer HTTP/JSON and Spring). Then **Week 4 review** and proof artifact **`week4`** (**#1–#5**, benchmark table in **`notes/week4.md`**, persisted todo API documented).

Do **not** start **ch.13–15**; the web and data path remains **Spring Boot / JPA** per **`CURRICULUM.md`**.

Do **not** pull **Sat** topics into today’s hands-on grading bar — if you only achieve **entity + repository + persist + list/create/get-by-id**, that is a successful **Fri**.
