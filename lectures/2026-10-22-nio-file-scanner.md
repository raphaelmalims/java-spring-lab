# Lecture 12 — NIO.2 file I/O: `Path`, `Files`, and `Scanner`

**Date:** Thu 22 Oct 2026 · **Book:** *Object-Oriented Programming and Java*, 2nd ed. — Ch.10 Input and Output Operations (pp.135–154) **context only** (Wed covered §10.1–10.4, 10.7–10.10); **today’s primary topics** are **`java.nio.file`** and **`Scanner`** per `CURRICULUM.md` Week 3 daily split — not additional printed § numbers.

**Time plan (1 h):** 10 min recap (Wed ch.10 streams & files) · 20 min read this lecture + Oracle *File I/O (NIO.2)* tutorial · 30 min hands-on (see §13 — curriculum **#1** and/or **#3**; problem statements in `CURRICULUM.md` only).

**Goal for today:** replace legacy **`java.io.File`** + ad-hoc stream wiring with **`Path`** and **`Files`** for common file tasks; use **`Scanner`** (and know when **`BufferedReader`** / **`Files.lines`** fit better); keep **exception placement** and **line-number reporting** from Wed while writing shorter, charset-explicit code — and preview how **“bad input in a file”** maps to **HTTP 400** logic on **Fri**.

> **Pacing note:** Week 3 in `CURRICULUM.md` schedules **Mon = ch.9 §9.1–9.5**, **Tue = §9.6 + try-with-resources**, **Wed = ch.10 streams & files**, **Thu = `java.nio.file` + `Scanner`** (this file), **Fri = Spring Boot "hello REST"**, **Sat = REST + error handling**. Do not invent schedule changes inside this lecture.

---

## 1. Recap hook (Wed 21 Oct — use the first 10 minutes)

Before NIO.2, from memory in `notes/`:

1. **Three bullets from yesterday:** e.g. **byte** vs **character** streams and when **#3** vs **#1** pick each; **`BufferedReader.readLine()`** vs end-of-file; why **`java.util.stream.Stream`** is not the same as an **`InputStream`**.
2. **One self-check from Lecture 11:** e.g. why **`FileNotFoundException`** differs from a parse error on line 42 in **#1**, or what buffered wrappers change for **#3** timing.
3. **Skim** **#1** / **#3** scratch — if you used **`FileReader`** or raw **`FileInputStream`**, note one place **`Path` + UTF-8** would have been clearer.

Wed taught the **classic I/O stack**. Thu is the **modern file API** most new Java code uses for paths, copying, and whole-file/line reads — still with **`IOException`** (checked) and try-with-resources where streams are opened manually.

*(Book ch.13–15 — GUI, applets, servlets — remain **skipped**; see `CURRICULUM.md` [Book chapters skipped on purpose](#book-chapters-skipped-on-purpose).)*

---

## 2. Why NIO.2 (`java.nio.file`) after `java.io` streams

The book (ch.10) centers **`InputStream` / `Reader`** because it predates the **NIO.2** file API (Java 7+). In this lab:

| Layer | Package / types | Typical job |
|---|---|---|
| **Stream read/write** (Wed) | `java.io` | Fine-grained **read/write loops**, binary copy (**#3** first pass), wrapping **`System.in`** |
| **File paths & helpers** (Thu) | `java.nio.file` — **`Path`**, **`Files`** | **Resolve paths**, **copy/move/delete**, **read/write strings or lines**, **newBufferedReader/Writer** with explicit charset |
| **Token / line parsing** (Thu) | `java.util` — **`Scanner`** | **Split** text into tokens, **`nextInt()`**, delimiter control; often over a **`Reader`** or **`InputStream`** |

**NIO** here means *New I/O* file support (`java.nio.file`), not “non-blocking sockets” (network NIO is out of scope for Week 3). **`Files.copy`** is the curriculum’s second implementation for **#3** — compare it fairly to your byte-stream copy (buffering, timing in `notes/`).

---

## 3. `Path` — location without `java.io.File`

**`Path`** is an interface; instances come from:

```java
Path path = Path.of("data", "numbers.txt");           // multi-segment, portable
Path abs   = Path.of("/tmp", "out.txt");              // platform-dependent root
Path fromUri = Path.of(URI.create("file:///..."));    // when needed
```

*(Older code uses **`Paths.get(...)`** — same idea; prefer **`Path.of`** in new code on Java 11+.)*

**What `Path` gives you (conceptual):**

| Operation | API idea | Notes |
|---|---|---|
| **Normalize / resolve** | `path.normalize()`, `base.resolve(relative)` | Build child paths without string concatenation bugs |
| **Name parts** | `getFileName()`, `getParent()`, `getRoot()` | Useful for logging “which file failed” |
| **Compare** | `equals` on path content vs **`Files.isSameFile`** for filesystem identity | Symlinks can surprise you — skim Oracle tutorial |
| **Check existence** | **`Files.exists`**, **`Files.isReadable`**, **`Files.isRegularFile`** | Prefer these over **`path.toFile().exists()`** |

**Link to exceptions:** many **`Files.*`** methods throw **`IOException`** (missing file, permission denied). **`NoSuchFileException`** / **`AccessDeniedException`** are subclasses — catch specifically only when your recovery differs; otherwise **`IOException`** at a boundary is fine.

**Working directory:** relative **`Path.of("foo.txt")`** resolves against the JVM’s current directory when the program runs — document that in **#1** test instructions (or use absolute paths in comments).

---

## 4. `Files` — static helpers for common file work

**`Files`** is a final class of **static methods** — no need to “open a Files object”. Group them by how they relate to Week 3 exercises:

### 4.1 Whole content (text)

| Method | Use when |
|---|---|
| **`Files.readString(path, charset)`** | Small/medium text file fits in memory; **#1** if you then split lines yourself |
| **`Files.writeString(path, content, charset, options...)`** | Write a report of bad lines from **#1** |
| **`Files.readAllLines(path, charset)`** | Need a **`List<String>`** with line terminators stripped |

Charset: **always** pass **`StandardCharsets.UTF_8`** (or a constant you define) for portable **#1** fixtures — unlike default **`FileReader`**.

### 4.2 Line-oriented streaming

| Method | Use when |
|---|---|
| **`Files.lines(path, charset)`** | Lazy **`Stream<String>`** over lines — **try-with-resources on the stream** (it wraps a **`Reader`**) |
| **`Files.newBufferedReader(path, charset)`** | Imperative loop; pairs with **`readLine()`** like Wed |

**#1 design (unchanged rules):** increment a **line number** as you consume lines (whether **`readLine`**, **`Files.lines`**, or **`Scanner`**). **`IOException`** usually aborts the whole read; **malformed number** on one line → report and **continue**.

### 4.3 Binary copy and metadata (**#3**)

| Method | Use when |
|---|---|
| **`Files.copy(source, target, CopyOption...)`** | **#3** second implementation; replaces manual byte pump |
| **`Files.size(path)`**, **`Files.getLastModifiedTime(path)`** | Sanity-check before/after copy in `notes/` |
| **`StandardCopyOption.REPLACE_EXISTING`** | Overwrite target when re-running experiments |

**Timing note:** **`Files.copy`** may use optimized transfer internally; comparing to an **unbuffered** **`FileInputStream`** loop is misleading — Wed’s pitfall still applies.

### 4.4 Creating and deleting (awareness)

**`Files.createDirectories`**, **`Files.delete`**, **`Files.deleteIfExists`** — useful when **#1** writes a summary file next to the input. Errors → **`IOException`**; decide checked custom exceptions (**#2**) vs plain propagation.

---

## 5. Try-with-resources with NIO streams

**`Files.newBufferedReader`** / **`Files.newBufferedWriter`** return **`Closeable`** resources:

```java
try (var reader = Files.newBufferedReader(path, StandardCharsets.UTF_8)) {
    // read loop
} catch (IOException ex) {
    // boundary handling
}
```

**`Files.lines`** returns a **`Stream<String>`** that **must** be closed:

```java
try (Stream<String> lines = Files.lines(path, StandardCharsets.UTF_8)) {
    lines.forEach(/* ... */);
}
```

Do not call **`Files.lines`** and forget **`close()`** — descriptor leaks match Tue’s lesson.

---

## 6. `Scanner` — parsing text from files or console

**`Scanner`** breaks a **`Readable`** source (often **`Reader`**, **`InputStream`**, or **`String`**) into **tokens** using **delimiters** (default: whitespace).

| Concern | Detail |
|---|---|
| **Construction** | `new Scanner(path, UTF_8)` (Java 15+ **`Scanner(Path, Charset)`**) or `new Scanner(Files.newBufferedReader(path, UTF_8))` |
| **Line vs token** | **`nextLine()`** reads through newline; **`nextInt()`** / **`next()`** skip delimiter-separated tokens — mixing them causes classic “skipped line” bugs |
| **End of input** | **`hasNextLine()`**, **`hasNextInt()`** before **`next*`** — otherwise **`NoSuchElementException`** (**unchecked**) |
| **Locale** | **`useLocale(Locale.US)`** if decimal formats matter — less critical for integer **#1** |
| **Regex delimiter** | **`useDelimiter("\\R")`** or **`"\\n"`** for line-like splitting — understand your test file’s line endings |

**When to prefer what for #1:**

| Approach | Strength | Watch out |
|---|---|---|
| **`Scanner` + `nextLine()`** per line, parse with **`Integer.parseInt`** | Simple line counter in your loop | Empty lines — decide if they are “bad” or ignored |
| **`Scanner` + `nextInt()`** in a loop | Compact for all-numeric files | Fails on non-numeric lines unless you catch and map to line numbers (harder — **`nextLine`** + parse is clearer for **#1**) |
| **`BufferedReader.readLine()`** / **`Files.lines`** | No token surprises | Same parse logic as Wed |

**`Scanner` implements `Closeable`** — try-with-resources when it wraps a file **`Reader`**.

**Console:** **`new Scanner(System.in)`** remains valid for small CLI tools; Week 3 proof set is **file**-oriented (**#1**, **#3**). Spring **Fri** uses HTTP, not stdin.

---

## 7. Formatting output (brief)

Ch.10 §10.8 discussed **`PrintWriter`** / **`printf`**. With NIO:

- **`Files.writeString`** or **`Files.newBufferedWriter`** + **`PrintWriter`** for human-readable bad-line reports.
- **`String.formatted`** / **`formatted`** (Java 15+) for messages without **`System.out`** in library code.

Keep **diagnostics on stderr** vs **results on stdout** if you split roles in **#1** (Wed self-check #6).

---

## 8. Relating Thu APIs to curriculum exercises

| Prompt | Thu angle | Exception angle |
|---|---|---|
| **#1** | **`Path`**, **`Files.newBufferedReader`** or **`Files.lines`**, or **`Scanner(Path, UTF_8)`**; explicit **UTF-8** | **`IOException`** at open/read boundary; per-line parse errors **inside** loop; **never crash** on bad lines |
| **#3** | Rewrite copy with **`Files.copy`**; compare to Wed byte-stream version | **`IOException`**; optional **`REPLACE_EXISTING`** |
| **#2** | Domain exceptions above I/O — e.g. checked “cannot write summary” vs unchecked “invalid token” | Same Mon/Tue rules |

**#4–#5** remain **Fri–Sat** (Spring Boot REST, centralized error handling).

---

## 9. Free resources (Week 3 — links only)

**Thu focus:** NIO.2 **`Path`/`Files`**, **`Scanner`**. Full Week 3 list lives in `CURRICULUM.md`; use these today:

- [Oracle Java Tutorials — File I/O (NIO.2)](https://docs.oracle.com/javase/tutorial/essential/io/fileio.html) (**primary read today**)
- [Oracle Java Tutorials — Basic I/O](https://docs.oracle.com/javase/tutorial/essential/io/) (Wed recap)
- [Oracle Java Tutorials — Scanning and Formatting](https://docs.oracle.com/javase/tutorial/essential/io/scanning.html) (**Scanner**)
- [Java SE 21 API — `java.nio.file.Files`](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/nio/file/Files.html)
- [Java SE 21 API — `java.util.Scanner`](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/Scanner.html)
- Baeldung: [Guide to java.nio.file](https://www.baeldung.com/java-nio-file)
- Baeldung: [Java try-with-resources](https://www.baeldung.com/java-try-with-resources) ( **`Files.lines`** )

*(Spring: **Fri** — [Building a RESTful Web Service](https://spring.io/guides/gs/rest-service/) per curriculum.)*

---

## 10. Glossary

**NIO.2** · **`Path`** · **`Path.of`** · **`Files`** · **`StandardCharsets.UTF_8`** · **`Files.readString`** · **`Files.writeString`** · **`Files.lines`** · **`Files.copy`** · **`CopyOption`** · **`StandardCopyOption.REPLACE_EXISTING`** · **`Files.newBufferedReader`** · **`Files.newBufferedWriter`** · **`Scanner`** · delimiter · **`nextLine`** · **`nextInt`** · **`hasNextLine`** · **`NoSuchElementException`** · **`IOException`** · **`NoSuchFileException`** · relative vs absolute path · try-with-resources on streams

---

## 11. Common pitfalls (Wed + Thu)

1. **Default charset** — **`FileReader`** / **`Scanner(path)`** without charset on JDK versions that default poorly; **#1** fixtures should be **UTF-8** and documented.
2. **`Files.lines` without try-with-resources** — stream leak.
3. **Mixing `Scanner.nextInt()` and `nextLine()`** — off-by-one line numbers in **#1** reports.
4. **Treating `NoSuchElementException` like a bad data line** — usually a logic bug (forgot **`hasNext*`**).
5. **Catching `IOException` per line** — I/O errors rarely mean “skip this line”; distinguish from parse errors (Wed pitfall #4).
6. **Unfair **#3** benchmark** — buffered byte copy vs **`Files.copy`** without noting buffer sizes and file size.
7. **String path concatenation** — `"dir/" + name` instead of **`dir.resolve(name)`** (portability and `..` segments).
8. **Assuming `Files.readString` for huge files** — memory pressure; use line stream or **`BufferedReader`** for large inputs.

---

## 12. Self-check (answer in `notes/`, then check Oracle tutorial / API docs)

1. What is the difference between **`Path.of("a", "b")`** and string concatenation `"a/b"`?
2. Name two **`Files`** methods you would use to implement **#3** without opening **`InputStream`** yourself.
3. For **#1**, why might **`nextLine()` + `Integer.parseInt`** be easier to align with **line numbers** than **`nextInt()`** alone?
4. Why must **`Files.lines`** appear in a try-with-resources block?
5. When **`Files.copy`** throws **`IOException`**, is that the same class of problem as a non-numeric line in **#1**?
6. What charset will you standardize on for Week 3 text exercises, and where do you pass it in the API?

*Stuck for more than 15 min? Write the exact error/question in `notes/` and ask for a review — hints and feedback, not solutions.*

---

## 13. Hands-on (Thu — curriculum prompts only; you write all code)

Week 3 proof set: **five prompts** in `CURRICULUM.md` Week 3 (**#1–#5**). Problem statements live in the curriculum only — **no solution code** in this lecture.

**Thu 22 Oct (after `java.nio.file` + `Scanner`):** per the suggested daily split, today’s focus is curriculum **#1** and/or **#3**, refactored or completed with **`Path`/`Files`** and/or **`Scanner`** where they genuinely simplify Wed’s approach.

### Curriculum **#1** (problem statement only)

Write a program that reads a text file of numbers (some lines intentionally malformed), sums the valid ones, reports each bad line with its **line number**, and **never crashes**. Decide **where to catch** and **where to propagate**.

### Curriculum **#3** (problem statement only)

Write a utility that **copies a file** using **try-with-resources**, then rewrite it with **`Files.copy`**. **Time both** on a large file and note differences (in `notes/`).

| Other prompts | Thu guidance |
|---|---|
| **#2** | Continue if incomplete — I/O boundaries should respect checked vs unchecked choices |
| **#4–#5** | **Fri–Sat** — Spring Boot REST and centralized error handling |

Work in scratch under e.g. `exercises/2026-10-22-nio-file-scanner/` until you consolidate into Week 3 proof paths per curriculum **Proof artifact** (`exercises/week3-todo-api/`, file-parsing exercise, `notes/week3.md`).

Closing habit (2 min): one line in `notes/` — what clicked / what confused you about **`Path`/`Files`** vs **`Scanner`**.

---

## 14. Bridge to Fri (per `CURRICULUM.md` Week 3)

**Tomorrow:** **Spring Boot “hello REST”** — generate a project (e.g. [start.spring.io](https://start.spring.io/)), add a **`@RestController`**, map **`GET`** (and later **POST**) endpoints, return **JSON** (records work well as DTOs). No servlets (book ch.15 skipped); the web stack is **Spring Boot** per curriculum.

**Conceptual link from Thu → Fri:**

| File / CLI (Thu) | HTTP (Fri onward) |
|---|---|
| Malformed line in **#1** → report line number, continue | Invalid request body → **400** with structured error (**Sat**) |
| Missing input file → **`IOException`** / your policy | Missing todo id → **404** (**#5**, Sat) |
| **`Files.readString`** / parse | **`@RequestBody`** + validation |

**Sat** adds **`@ExceptionHandler`** / **`@RestControllerAdvice`** so failure classification is centralized — same discipline as “where do I catch **`IOException`** vs parse errors?” today.

Do not jump to Swing (ch.13) or applets (ch.14); stay on the curriculum path through **REST + error handling** this week.
