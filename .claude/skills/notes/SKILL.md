---
name: notes
description: Generate STUDY.md — an evidence-based study guide of this codebase with file:line references, concept walkthrough, honest weaknesses, likely live extensions, and a self-quiz. Run before an interview that extends this repo; read-only.
---

Produce a STUDY GUIDE of this repository, written to STUDY.md, to prepare the human to
extend this codebase live and explain every decision out loud.

Rules: **READ-ONLY** — change no code. Every claim gets a file:line reference. Quote
short fragments (≤10 lines), never whole files. If anything looks broken on a clean
clone (missing env, failing tests, install errors), flag it at the very top as
**PRIORITY ZERO** before anything else.

Sections, in order:

1. **THIRTY-SECOND MAP** — what this service does, then the full request lifecycle of
   the primary endpoint: route → handler → business rules → data access → response, as
   a numbered chain where every step is file:line.

2. **LAYER INVENTORY** — where each layer actually lives: HTTP/handler, domain rules,
   storage. If layers are blurred (rules inside handlers, SQL inside services), say so
   bluntly — report the real seams, not the ideal ones.

3. **CONCEPT WALKTHROUGH** — for each concept: where it lives (file:line + fragment),
   the pattern used, and one sentence the human can say out loud about why. If absent,
   write "NOT PRESENT" and name the exact file:line where it would go:
   a. Input validation at the boundary
   b. Error handling (expected/unexpected split, error shape, every swallowed catch)
   c. Core domain rules — enumerate EVERY rule/branch, implementation, and test if any
   d. API design (endpoints, status codes, pagination, DTO vs raw rows)
   e. Persistence (schema, migrations, query construction, indexes)
   f. Tests (coverage; rank the top 5 untested risks)
   g. Idempotency & race conditions — every check-then-act write path, line-precise
   h. Logging/observability

4. **HONEST WEAKNESSES** — the 5 things a strong reviewer pokes at first: file:line,
   why the criticism is fair, and the one-line answer that owns it and names the fix.

5. **LIKELY LIVE EXTENSIONS** — 7 plausible new requirements for a service of this
   shape. For each: the seam it lands in (file:line), a 3–5 step plan with a verifiable
   outcome per step, the first test to write, and the one trap (race, N+1, timezone,
   partial failure) hiding inside it.

6. **QUIZ ME** — 10 interviewer-style questions about THIS code, hardest first, with
   answers in a separate section at the bottom for self-testing.
