# java-spring-lab

A hands-on Java study lab: from a solid Java refresher to Spring Boot and Java-based ML/AI systems.

Owner: Raphael Malims. Study time: 1 hour/day, Mon–Sat, starting **Mon 5 Oct 2026**.
Course text: *Object-Oriented Programming and Java*, 2nd ed. (Poo, Kiong, Ashok). The book is **not** in this repo (copyright).

## Purpose

Become a confident Java professional: core language, when to use classes vs records vs interfaces vs static functions, generics, collections, JVM memory (heap/stack/GC), concurrency, modern Java (records, `var`, streams, lambdas, sealed types, virtual threads), then Spring Boot (REST, JPA, testing, calling an LLM) and a small service in front of the PersonaLearn RAG endpoint.

## Layout

| Path | What lives there |
|---|---|
| `CURRICULUM.md` | **Source of truth** for the ~10-week plan (+ buffer week), exercise prompts and proof artifacts |
| `lectures/` | One Markdown lecture per study block, e.g. `2026-10-05-ch1-3-oop-basics.md` |
| `exercises/` | **Raphael's own code.** Nothing here is written by an agent |
| `notes/` | Personal notes, review feedback, cheat-sheets |

## Rules

1. **Raphael writes ALL exercise code himself.** No pasted or generated solutions.
2. Agents (AI assistants) may only write **lectures, notes, and reviews** (comments/feedback on code he wrote). They never write exercise solutions and never edit `exercises/`, except its README.
3. **NEVER commit secrets** — API keys, tokens, passwords, or `.env` files. `.env` is git-ignored; use `.env.example` with placeholder values instead.
4. Commit author: `Raphael Malims <raphaelmalimsj@gmail.com>`.
5. Small, frequent commits with short messages (e.g. `ch1-3 exercises: Counter + BankAccount`).

## Daily 1-hour block

10 min recap → 20 min lecture read → 30 min hands-on. Details in `CURRICULUM.md`.
