# Lecture 11 — Input and output: byte vs character streams, files, and the I/O stack

**Date:** Wed 21 Oct 2026 · **Book:** *Object-Oriented Programming and Java*, 2nd ed. — Ch.10 Input and Output Operations (pp.135–154; read **§10.1–10.4** and **§10.7–10.10** today; **skim §10.5–10.6** and **§10.11** per `CURRICULUM.md` Week 3)

**Time plan (1 h):** 10 min recap (Tue ch.9 §9.6 + try-with-resources) · 20 min read this lecture + assigned book sections · 30 min hands-on (see §11 — curriculum **#1** and/or **#3**; problem statements in `CURRICULUM.md` only).

**Goal for today:** see how Java models **I/O as streams**; choose **byte** vs **character** streams for the job; open and read **files** with the classic `java.io` stack while **closing** via try-with-resources; connect **`IOException`** and handler placement to curriculum **#1** (line-level parse errors) and **#3** (byte copy) — without confusing this week’s **Stream API** (`java.util.stream`) with **I/O streams**.

> **Pacing note:** Week 3 in `CURRICULUM.md` schedules **Mon = ch.9 §9.1–9.5**, **Tue = §9.6 + try-with-resources**, **Wed = ch.10 streams & files** (this file), **Thu = `java.nio.file` + `Scanner`**, **Fri = Spring Boot "hello REST"**, **Sat = REST + error handling**. Do not invent schedule changes inside this lecture.

---

## 1. Recap hook (Tue 20 Oct — use the first 10 minutes)

Before reading ch.10, from memory in `notes/`:

1. **Three bullets from yesterday:** e.g. why **`finalize()`** is not a substitute for **`close()`**; **`AutoCloseable`** / **`Closeable`** and try-with-resources **close order** (reverse of declaration); **suppressed exceptions** when both body and `close()` fail.
2. **One self-check from Lecture 10:** e.g. what you would **measure** for curriculum **#3**, or name two **`AutoCloseable`** types you expect this week.
3. **Skim** curriculum **#2** / **#3** scratch — if you copied a file Tue, which stream types did you use (`InputStream`/`OutputStream` vs `Reader`/`Writer`)? Note one **`IOException`** you handled vs propagated.

Tue taught **resource safety**. Wed names the **stream types** those resources usually are — so **#1** (text numbers + bad lines) and **#3** (binary copy) pick the right abstractions.

*(Book ch.13–15 — GUI, applets, servlets — remain **skipped**; see `CURRICULUM.md` [Book chapters skipped on purpose](#book-chapters-skipped-on-purpose).)*

---

## 2. Two different “streams” (disambiguation)

| Name in conversation | Package / type | Role |
|---|---|---|
| **I/O stream** (today) | `java.io` — `InputStream`, `Reader`, … | **Sequential bytes or characters** from/to files, sockets, console |
| **Stream API** (Week 2) | `java.util.stream` — `Stream<T>` | **Pipeline over in-memory collections** (`filter`, `map`, `collect`) |

Curriculum **#1** can use **either** a **`Reader`** loop **or** **`Files.lines`** (Thu); both are valid once you know what an **`IOException`** means at open/read/close time. Do not use **`Stream<T>`** alone to read a file from disk — you still need an I/O source behind it.

---

## 3. I/O model (§10.1; pp.135–137)

**Input** moves data **into** your program; **output** moves data **out**. The book presents the **stream** metaphor: data arrives as a **sequence** (bytes or characters), read or written **incrementally**, not necessarily all at once in memory.

Core ideas (paraphrase §10.1):

| Idea | Meaning |
|---|---|
| **Source / sink** | Input has a **source** (file, keyboard, network); output has a **sink** (file, screen, socket). |
| **Abstraction** | Your logic calls **`read`** / **`write`** (or helpers like **`Scanner`**) on a stream type; the concrete class hides OS details. |
| **Errors** | Missing files, permission denied, disk full → typically **`IOException`** (**checked** — Mon/Tue). |
| **Closing** | Streams hold OS resources; **try-with-resources** (Tue) applies to most stream classes implementing **`Closeable`**. |

**Link to exceptions:** opening a non-existent path for read often throws **`FileNotFoundException`** (subclass of **`IOException`**). For **#1**, decide: catch at the boundary and print a friendly message, or declare **`throws`** and let **`main`**’s caller fail — curriculum asks you to **never crash** on bad **lines**, not necessarily to hide a **missing file** (your design choice — document it in `notes/`).

---

## 4. Byte streams vs character streams (§10.2–10.3; pp.137–142)

Java splits the classic I/O hierarchy into two parallel families:

```
java.io
├── InputStream  ──► FileInputStream, BufferedInputStream, …
├── OutputStream ──► FileOutputStream, BufferedOutputStream, …
├── Reader       ──► FileReader, BufferedReader, …
└── Writer       ──► FileWriter, BufferedWriter, …
```

| | **Byte streams** | **Character streams** |
|---|---|---|
| **Unit** | Raw **8-bit bytes** | **16-bit char** units (Unicode) |
| **Typical use** | Binary data: images, copied files (**#3**), compressed archives | **Text**: line-oriented files, config, logs (**#1**) |
| **Encoding** | No charset — you interpret bytes yourself | **`Reader`/`Writer`** use a **charset** (platform default unless you specify **`InputStreamReader`/`OutputStreamWriter`** with **`StandardCharsets.UTF_8`**) |
| **Book emphasis** | **`InputStream` / `OutputStream`** hierarchy | **`Reader` / `Writer`** hierarchy |

**Rule of thumb for Week 3:**

- **#3** (file copy, compare timing with **`Files.copy`**) → **byte** streams or NIO **`Files.copy`** (both treat file as opaque bytes).
- **#1** (lines of text that should parse as numbers) → **character** streams or **`BufferedReader.readLine()`**, or **`Scanner`** over a **`Reader`** — **Thu** adds **`Scanner`** and **`Files.readString`** / line streams explicitly.

**Bridge:** a **character stream** is often wrapped around a **byte stream** plus a **charset** (`InputStreamReader`). The book’s §10.3 explains why text needs a separate branch — binary-safe copy (**#3**) should not accidentally corrupt data by treating bytes as chars.

---

## 5. Standard I/O and basic file classes (§10.4; pp.142–145)

Programs commonly attach to:

| Stream | Typical static fields | Notes |
|---|---|---|
| **Standard input** | `System.in` (`InputStream`) | Often wrapped as `new Scanner(System.in)` for text |
| **Standard output** | `System.out` (`PrintStream` — byte-based but prints text) | **`println`** for human-readable output |
| **Standard error** | `System.err` | Same idea as stdout; use for diagnostics |

**File-backed byte streams (book §10.4):**

- **`FileInputStream` / `FileOutputStream`** — open a path for byte read/write.
- Construct with **`File`** or **`String`** path (legacy **`java.io.File`**; **Thu** prefers **`java.nio.file.Path`**).

**File-backed character streams:**

- **`FileReader` / `FileWriter`** — convenience for **text** using the **default charset** (fragile for portable apps).
- **Modern habit:** `Files.newBufferedReader(path, UTF_8)` (**Thu**) or **`InputStreamReader`** with an explicit charset when staying on **`java.io`**.

**Try-with-resources shape (today’s default):**

```java
try (Reader reader = new FileReader(path)) {
    // read and parse
} catch (IOException ex) {
    // log, wrap with cause, or propagate — Mon §6.3
}
```

Replace **`FileReader`** with explicit UTF-8 when you control file format (**#1** test files should use a charset you document in a comment).

---

## 6. Buffered and data streams — skim (§10.5–10.6; pp.145–148)

**§10.5 — Buffered streams:** **`BufferedInputStream`**, **`BufferedOutputStream`**, **`BufferedReader`**, **`BufferedWriter`** reduce system calls by reading/writing **chunks** into an in-memory buffer. For **#3** on a **large file**, wrapping raw file streams in **buffered** streams often changes throughput — a fair comparison when you **time** manual copy vs **`Files.copy`**.

**§10.6 — Data streams:** **`DataInputStream` / `DataOutputStream`** read/write primitive binary types in a defined order. Useful for **binary protocols**, not for **#1** text numbers. **Skim** the API idea; you rarely need them in Spring-era application code unless you own a binary format.

---

## 7. Readers, writers, print streams, random access (§10.7–10.10; pp.148–153)

### 7.1 Readers and writers (§10.7)

**`Reader` / `Writer`** abstract character input/output. **`BufferedReader.readLine()`** is the workhorse for **line-based text** until **`Files.lines`** (Thu):

- Returns **`null`** at end-of-stream.
- Line **content** excludes platform line terminators — good for parsing numbers per line in **#1**.
- **`IOException`** on read — distinguish from **malformed number** (your **`NumberFormatException`** or custom handling **inside** the loop).

**Design for #1:** keep **I/O failures** (cannot read file) separate from **data failures** (line 17 is not a number). Report **line number** (increment a counter each **`readLine`**) and **continue** summing valid lines.

### 7.2 Print streams and print writers (§10.8)

**`PrintStream`** / **`PrintWriter`** add **formatted, convenient output** (`print`, `println`, `printf`). **`System.out`** is a **`PrintStream`**. For file output of **reports** (e.g. listing bad lines from **#1**), **`PrintWriter`** on a **`Writer`** or **`Files.newBufferedWriter`** (Thu) keeps text encoding explicit.

### 7.3 Random access files (§10.9)

**`RandomAccessFile`** can read/write at a **seek** position — useful for indexed binary files. **Skim** for awareness; Week 3 exercises do not require it. Prefer sequential streams or NIO for normal text/binary copy.

### 7.4 Serialization preview (§10.10)

The book introduces **object serialization** — writing object graphs to byte streams. **Week 4** skims **ch.16** at a higher level; **prefer HTTP/JSON and Spring** for interchange. Today: know that **byte streams** can carry **structured binary**, but your **todo API** and **#1** stay **text** or plain **byte copy**.

---

## 8. Other topics — skim (§10.11; p.154)

§10.11 typically collects **miscellaneous I/O** (console nuances, deprecated APIs, or legacy patterns in a 2008 text). **Skim** headings only; do not invest time in APIs the JDK has superseded. **Thu** **`java.nio.file`** is the modern file path for new code in this lab.

---

## 9. Relating stream choice to curriculum exercises

| Prompt | Stream angle | Exception angle |
|---|---|---|
| **#1** | **Character**-oriented read; line loop; optional **`Scanner`** Thu | **`IOException`** at open/read; per-line parse errors **caught inside loop**; program continues |
| **#3** | **Byte** **`InputStream`/`OutputStream`** (+ buffered wrappers); then **`Files.copy`** | **`IOException`** on copy; try-with-resources on both streams Tue |

**#2** (custom checked/unchecked) may appear in domain logic **above** I/O — e.g. unchecked for “line empty”, checked for “summary file cannot be written”. Stay consistent with Mon’s rationale.

---

## 10. Free resources (Week 3 — links only)

**Wed focus:** byte vs character streams, files, classic I/O. Full Week 3 list lives in `CURRICULUM.md`; use these today:

- [Oracle Java Tutorials — Basic I/O](https://docs.oracle.com/javase/tutorial/essential/io/)
- [Oracle Java Tutorials — I/O Streams](https://docs.oracle.com/javase/tutorial/essential/io/streams.html) (byte vs character overview)
- [Oracle Java Tutorials — File I/O (NIO.2)](https://docs.oracle.com/javase/tutorial/essential/io/fileio.html) (preview — **Thu** primary)
- Baeldung: [Exception Handling in Java](https://www.baeldung.com/java-exceptions) (I/O + wrap with cause)
- Baeldung: [Java try-with-resources](https://www.baeldung.com/java-try-with-resources) (Tue recap on streams)

*(Spring error handling: **Fri–Sat** per curriculum daily split.)*

---

## 11. Glossary

I/O stream · source · sink · **`InputStream`** · **`OutputStream`** · **`Reader`** · **`Writer`** · byte stream · character stream · charset · **`FileInputStream`** · **`FileReader`** · **`BufferedReader`** · **`readLine`** · **`IOException`** · **`FileNotFoundException`** · **`PrintWriter`** · **`RandomAccessFile`** (skim) · standard input/output/error · binary vs text · line number (logical, not `RandomAccessFile`)

---

## 12. Common pitfalls (Mon + Tue + Wed)

1. **Using `FileReader` without knowing the charset** — mojibake on non-ASCII test data (**#1**).
2. **Parsing numbers with byte streams** — reading raw bytes into **`String`** manually instead of **`Reader`** / **`Scanner`**.
3. **Confusing `java.util.stream.Stream` with file I/O** — Week 2 API vs **`java.io`** (§2 above).
4. **Catching `IOException` inside the line loop and treating it like a bad line** — usually wrong; I/O errors often mean stop or abort the whole read.
5. **Swallowing parse errors** — **#1** requires **reporting** bad lines with **line numbers**, not silent skip without trace.
6. **Forgetting buffered wrappers on large **#3** copies** — unfair timing vs **`Files.copy`** (which buffers internally).
7. **Not closing streams** — Tue’s try-with-resources rule; still applies to every **`new FileInputStream`** today.
8. **Wrapping without cause** — `throw new RuntimeException("read failed")` losing **`IOException`** detail (Mon).

---

## 13. Self-check (answer in `notes/`, then check against the book / Oracle tutorial)

1. When should you prefer **`InputStream`** over **`Reader`** for a given task?
2. What does **`BufferedReader.readLine()`** return at end-of-file?
3. Why is **`FileNotFoundException`** usually handled differently from a **`NumberFormatException`** on line 42 in **#1**?
4. Name one advantage of wrapping a file **`InputStream`** in **`BufferedInputStream`** before copying.
5. How does **`Files.copy`** (preview) relate to the byte-stream copy in **#3**?
6. What is the difference between **standard output** and **standard error**, and when would you print a bad-line report to each?

*Stuck for more than 15 min? Write the exact error/question in `notes/` and ask for a review — hints and feedback, not solutions.*

---

## 14. Hands-on (Wed — curriculum prompts only; you write all code)

Week 3 proof set: **five prompts** in `CURRICULUM.md` Week 3 (**#1–#5**). Problem statements live in the curriculum only — **no solution code** in this lecture.

**Wed 21 Oct (after ch.10 §10.1–10.4, 10.7–10.10):** per the suggested daily split, today’s focus is curriculum **#1** and/or **#3**. Use **byte** streams for **#3** and **character**/line-oriented reading for **#1**.

### Curriculum **#1** (problem statement only)

Write a program that reads a text file of numbers (some lines intentionally malformed), sums the valid ones, reports each bad line with its **line number**, and **never crashes**. Decide **where to catch** and **where to propagate**.

### Curriculum **#3** (problem statement only)

Write a utility that **copies a file** using **try-with-resources**, then rewrite it with **`Files.copy`**. **Time both** on a large file and note differences (in `notes/`).

| Other prompts | Wed guidance |
|---|---|
| **#2** | Finish if Tue left it incomplete — exception choices should still make sense beside I/O |
| **#4–#5** | **Fri–Sat** — Spring Boot REST and centralized error handling |

Work in scratch under e.g. `exercises/2026-10-21-ch10-streams-files/` until you consolidate into Week 3 proof paths per curriculum **Proof artifact** (`exercises/week3-todo-api/`, file-parsing exercise, `notes/week3.md`).

Closing habit (2 min): one line in `notes/` — what clicked / what confused you about **byte vs character** streams or **line-level** error reporting.

---

## 15. Bridge to Thu (per `CURRICULUM.md` Week 3)

**Tomorrow:** **`java.nio.file`** — **`Path`**, **`Paths`**, **`Files`** (`readString`, `writeString`, `lines`, `copy`, `newBufferedReader`/`newBufferedWriter`) plus **`Scanner`** for parsing. You will rewrite parts of **#1** and **#3** with shorter, more portable APIs while keeping the same **exception** and **line-number** behavior.

**Later this week:** **Fri** first **`@RestController`**; **Sat** maps failures to **HTTP** status codes and **`@RestControllerAdvice`**. Text file parse errors (**#1**) foreshadow **400** responses; missing resources foreshadow **404** — same “classify the failure” muscle as Mon **#2**.

Do not jump to servlet chapters (book ch.15 skipped) or Swing (ch.13); the web path stays **Spring Boot** per `CURRICULUM.md`.
