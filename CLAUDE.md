# CLAUDE.md — Interview Working Contract

This file has two parts. **Part A is a contract**: hard rules the agent follows, enforceable
at specific decision points. **Part B is a field guide**: engineering practices with their
reasoning, for the human to review and articulate live. Part B informs judgment calls;
Part A is never overridden by it.

This is the stack-agnostic core. The only section edited per-repo is the **TUNING LAYER**
at the bottom.

---

## PART A — THE CONTRACT

### A0. Session start: declare the mode

Before anything else, establish which mode we are in. If the human hasn't said, ask.

- **MODE: EXTEND** — an existing codebase. Prime directive: *conform*. Read before you
  write. Existing conventions beat your preferences, even when yours are better.
- **MODE: GREENFIELD** — empty repo. Prime directive: *smallest structure that carries the
  first slice*. No speculative architecture; every file must earn its existence today.

Why this matters: the same rule ("good structure") means opposite things in each mode.
In EXTEND, adding your favorite validation library to a repo that already has one is a
failure. In GREENFIELD, spending ten minutes on folder taxonomy before any code runs is
a failure. Declaring the mode makes the prime directive explicit instead of vibes.

**Triggers:** preferred — the `/survey` (EXTEND) or `/scaffold` (GREENFIELD) skill
(`.claude/skills/`), which also runs that mode's startup checklist. Fallback when skills
weren't copied in (e.g. dictated setup): the human types "MODE: EXTEND" or "MODE:
GREENFIELD" and the same checklist runs from the MODE PLAYBOOKS section below.

### A1. Plan first — no code before an approved plan

Never create or modify code before a written plan exists AND the human has approved it.

- Scale the plan to the work: a feature gets PLAN.md; a small change gets 2–3 bullets in
  chat. But the sequence is invariant: plan → approval → code.
- A plan step is only valid if it names a **verifiable outcome** ("endpoint returns 200
  with filtered list", not "work on endpoint").
- If the human says "just do it", restate the plan in one line and proceed — the point is
  a shared mental model, not ceremony.

*Trigger moment: about to run the first Write/Edit. If no approved plan exists, stop.*

### A2. One vertical slice at a time

Build depth-first: one path working end-to-end (request → logic → storage → response)
before starting a second path. Never leave more than one slice half-built.

- A slice is DONE when it runs and its core logic has a test — not when the code exists.
- When time pressure hits, a finished slice is demoable; six half-built layers are not.

*Trigger moment: about to start a new file/feature while the current slice has no passing
run. Finish or explicitly park (announce it) first.*

### A3. Announce major changes before making them

The following require a one-line proposal and human OK before doing them:
new dependency · schema/migration change · new module or top-level file · change to a
public API shape · deleting or rewriting anything that exists · any global refactor.

Everything else (implementing the approved plan step) proceeds without asking.
The line exists so the human is never surprised by the diff — surprises in review are
the signal that the human lost control of the session.

### A4. Every diff is reviewed before it's "done"

After each change, present what changed (per the output contract, A6) and wait for the
human's review before treating the step as complete. Nothing lands in the demo path
unreviewed. If the human waves it through, that's their call — but the gate always fires.

Rationale: the human is accountable for every line on screen. "The AI wrote it" is not
a defense in an interview or in production.

### A5. No slop

Never present code containing: dead code, unused imports/abstractions, stub functions
without a stated reason, swallowed errors (`catch` that ignores), copy-paste duplication,
speculative generality (interfaces with one implementation, config for things that don't
vary), or comments narrating what the code obviously does. Generated code gets the same
bar as hand-written. If a shortcut is deliberate (time pressure), name it out loud and
put it in NEXT — a *named* shortcut is judgment; a silent one is slop.

### A6. Output contract — "caveman mode"

Every work update — a completed step, a diff to review, a proposal under A3 — is reported
in exactly this skeleton. Plain text labels, one line each, max 5 lines total:

```
CHANGED: what is different now (one line; "nothing yet" for proposals)
WHY: the requirement or decision this serves (one line)
FILES: paths touched, comma-separated ("none" if none)
REVIEW: the ONE thing needing human judgment right now
NEXT: next step, or the named shortcut being deferred
```

Rules:
- All five labels, always, in this order — even when a line is trivial. A fixed shape is
  what makes it scannable; variation defeats the purpose.
- One line per label. If a line wants to be a paragraph, it becomes a one-line flag in
  REVIEW and waits to be asked about.
- No prose before or after the block during build flow. Full prose only on explicit
  request — preferred trigger: the `/why` skill; fallback phrases: "explain",
  "teach", "go deep", "why". Afterwards, return to caveman mode (`/terse` snaps back
  immediately if output drifts into prose).
- REVIEW is never empty and never "looks good". If nothing needs judgment, say what was
  verified instead ("tests pass; no decision needed").

Rationale: mid-build, the human is narrating to an interviewer and can spare ~3 seconds
per update. A fixed skeleton means the eyes know where to look; REVIEW preserves the A4
control point; NEXT is where A5's named shortcuts live so they are never silent.

### A7. Ambiguity protocol

When requirements are ambiguous: (1) state the interpretation chosen, (2) prefer the
**reversible** option, (3) record the assumption where the human will see it. Never
silently guess. In an interview, the assumption gets said out loud — stated assumptions
are evaluated as judgment; discovered ones are evaluated as bugs.

### A8. Tests on core logic

Domain rules — eligibility, money, state transitions, anything with branches — get tests
with or before the implementation. Glue (routing, serialization, config) does not, unless
time allows. See Part B §6 for the reasoning and the exceptions.

### A9. Parallelism doctrine — fan out to read, serialize to write, fan out to review

Subagents are encouraged for **reading** (exploration, research, `/map`) and for
**reviewing** (fresh-context diff review via the `diff-reviewer` agent). The **write
stream stays single**: one session (this one) makes all code changes on the demo path.
At most one background delegation of a truly independent artifact (tests against an
agreed interface, docs), and only with the human's OK (A3).

Rationale: reads are embarrassingly parallel and protect the main context window;
parallel writers move the bottleneck to human review — and in an interview, the
reviewer is the interviewer. A4's gate only works with one diff stream to review.

---

## PART B — FIELD GUIDE

Ten practices. Each entry: what it is → why it works (the mechanism, not the slogan) →
**universal or situational** → when it applies → when it's overkill → how to say it live.
The universal/situational tag is the part interviewers probe: applying a practice where
it doesn't pay is scored the same as missing it where it does.

### B1. Idempotency — *situational technique, universal question*

**What:** an operation applied twice has the same effect as once.
**Why it works:** every effectful operation that *can* be re-attempted eventually *will*
be — network retries, double-clicks, at-least-once queue delivery, webhook redelivery.
Idempotency converts "retry" from a dangerous action into a safe one, which is what makes
reliable systems out of unreliable networks.
**Applies when:** writes reachable by external callers or retry machinery: bookings,
payments, webhook handlers, queue consumers. Canonical: booking a shift — two clicks must
not double-book.
**Overkill when:** pure reads (already idempotent); internal one-shot scripts; and
usually, building idempotency-*key* infrastructure when **natural idempotency** is
available — a `PUT` with full state, or a DB unique constraint, gives you the property
for free. Keys are the upgrade for operations that aren't naturally idempotent or where
you must distinguish "retry" from "conflict".
**Say it live:** "Booking is a retriable write, so it must be idempotent. Cheapest
correct version: unique constraint on (worker_id, shift_id) and treat the violation as
'already booked'. An idempotency-key table is the upgrade path if we need to tell retries
apart from genuine conflicts."

### B2. Input validation at boundaries — *universal*

**What:** parse external input (HTTP body, file row, queue message, LLM output) into a
typed domain object at the exact point it enters the system. Inside that boundary, trust
the types; never re-validate.
**Why it works:** "parse, don't validate." One checkpoint means the entire interior can
assume well-formed data — every function signature becomes a guarantee instead of a hope,
and malformed input fails *at the edge, with context* (which row, which field) instead of
three layers deep as a mystery `undefined`.
**Applies when:** always, at every trust boundary. The situational part is only *depth*:
shape/type checks at the edge always; cross-field *business* rules (can this worker take
this shift?) belong in the domain layer, not the schema.
**Overkill when:** re-validating between your own internal layers, or validating reads
from your own database — that's validating data you already guaranteed, pure noise.
**Say it live:** "This zod/Pydantic schema is the trust boundary. Everything past this
line takes typed input and doesn't re-check — that's why validation appears exactly once
in this codebase."

### B3. Error handling & failure modes — *universal principle, situational depth*

**What:** classify failures and treat each class deliberately. Expected failures (bad
input, not-found, conflict) → typed results / 4xx with a useful message. Unexpected
failures (bugs, down dependencies) → fail fast, loud, logged with context → 5xx.
**Why it works:** the most expensive bug class is the swallowed error — a failure that
produces no signal, so the system keeps running on corrupt assumptions. Fail-fast turns
"corrupt state discovered next week" into "stack trace now."
**Applies when:** always. The classification (whose fault, can the caller act on it,
what do we log) is the universal part.
**Overkill when:** `try/catch` wrapping every function (catch where you can *handle or
add context*, nowhere else); retries/circuit-breakers without a flaky dependency to
justify them; a bespoke exception hierarchy for a 90-minute build — one `AppError` with
a `kind` field covers it.
**Say it live:** "My split: expected failures get typed results the caller can act on;
unexpected ones crash loudly. I never catch-and-continue silently — a swallowed error is
strictly worse than a crash, because a crash tells you where."

### B4. Separation of concerns — *universal direction, situational depth*

**What:** minimum viable layering: **handler → domain → storage**. The domain layer —
the actual business rules — stays pure: no framework imports, no I/O, plain functions on
plain data.
**Why it works:** two mechanisms. (1) Testability: pure rules test in milliseconds with
no DB or HTTP scaffolding — this is what makes rule A8 cheap to obey. (2) Change
isolation: frameworks and storage change for different reasons than business rules;
separating them means a schema change can't silently alter eligibility logic. Interview
bonus: the rules become one readable screen the interviewer can follow.
**Applies when:** always, at this 3-layer minimum — it costs nearly nothing.
**Overkill when:** hexagonal ports-and-adapters, DI containers, repository interfaces
"so we could swap Postgres" (you won't) — abstraction ahead of a demonstrated second
implementation is speculation, and speculation is slop (A5).
**Say it live:** "Eligibility is a pure function — no framework, no DB handle. I can
unit-test every branch in milliseconds, and you can read the whole business policy on
one screen."

### B5. Reconciliation & consistency for data flows — *situational, but likely in-domain*

**What:** when data crosses a system boundary (file → DB, vendor feed → your tables,
service ↔ service), build the accounting loop: every input row ends as exactly one of
**loaded / rejected-with-reason / quarantined**, and totals are compared in vs out.
**Why it works:** cross-system transfers fail *partially* — row 40,001 of 80,000. Without
reconciliation the failure is silent and you learn about it from a customer or a
regulator. The count-comparison converts "hope it all arrived" into a checkable
invariant. This is the regulated-finance instinct, and healthcare/ICHRA data has the
same audit posture.
**Applies when:** ingesting external data, syncing systems, anything involving money
or member/eligibility records. In a greenfield interview with a "messy file" problem,
the reject/quarantine path IS the differentiating feature — plan it as its own slice.
**Overkill when:** single-system CRUD with no external feeds — there's no boundary to
reconcile across.
**Say it live:** "Every input row is accounted for: loaded, rejected with a reason, or
quarantined for review. The import summary — N in, N accounted — is the contract. Silent
drops are the failure mode that gets you fined."

### B6. Testing strategy — *universal that core logic is tested; shape is situational*

**What:** under time pressure, a deliberate triage: (1) unit tests on domain rules —
the branching logic; (2) one integration test on the primary endpoint's happy path;
(3) consciously skip the rest and say so.
**Why it works:** tests buy regression-safety per minute spent, and the ROI concentrates
where behavior *branches*. An eligibility function with eight conditions is eight
opportunities to be wrong; a route registration is one line of glue. Tests on rules also
enable fearless refactoring later in the session — which you'll need when the interviewer
adds a requirement.
**Applies when:** always for rules with branches, money math, state machines, parsers.
**Overkill when:** mock-heavy controller tests (they test the mocks), 100% coverage as a
goal, E2E suites in an interview, tests written after the fact just to have them.
**Say it live:** "I'm testing the rules table, not the plumbing — eligibility has eight
branches and that's where the bugs live. The HTTP layer gets one integration test proving
the wiring."

### B7. Observability & logging — *universal minimum, situational depth*

**What:** minimum: structured (key-value) logs at **decisions and failures**, always
carrying the relevant IDs (request, worker, shift). Depth (metrics, tracing, dashboards)
is added when there's an operator to consume it.
**Why it works:** you debug production with only what you chose to record beforehand —
logging is a bet placed before the incident. Logging *decisions* ("worker 12 excluded:
credential expired") captures the why; logging *motion* ("entering function") captures
noise. In a live session it also pays immediately: when something misbehaves, your own
logs are your fastest debugger.
**Applies when:** always, at the minimum level — it's ~one line per decision point.
**Overkill when:** metrics/OTel/dashboards in an interview; log-every-call noise;
logging PII in a regulated domain (log IDs, never payloads, in healthcare/fintech).
**Say it live:** "I log decisions, not motion — one line saying *why* worker 12 was
excluded beats ten lines of 'entering function'. IDs only, no payloads: this is PHI."

### B8. API design — *universal basics, situational elaboration*

**What:** resources named as nouns, verbs from HTTP methods, correct status-code
families (400 caller / 404 missing / 409 conflict / 500 us), one consistent error shape,
pagination on any unbounded collection, and responses mapped through a DTO — never raw
DB rows.
**Why it works:** an API is a contract with someone who can't read your code. Predictable
conventions mean the caller can guess correctly, and status codes are machine-readable
failure semantics (retry a 500; don't retry a 400). The DTO mapping is a security and
compatibility boundary: raw rows leak columns and freeze your schema.
**Applies when:** always — these basics cost nothing extra at write time.
**Overkill when:** versioning on day one (defer until a real consumer exists — but be
*able to say* how you'd add it), HATEOAS, cursor pagination where offset serves the demo.
**Say it live:** "Available-shifts is a filtered query on a collection: GET with query
params, paginated from day one because the shift list is unbounded, 409 on booking
conflicts so a client can distinguish 'retry' from 'give up'."

### B9. Database migrations — *universal once a schema exists*

**What:** every schema change is a versioned, ordered, replayable file in the repo,
applied by a tool — never by hand-typed SQL in a console.
**Why it works:** the database schema is code that lives outside the repo unless you
force it in. Migrations make the schema reproducible — clean clone + migrate = working
system — which is exactly the "it didn't run on their machine" failure the runbook
(artifact #4) exists to prevent. History also documents *when and why* the schema
changed.
**Applies when:** from the first table, even greenfield in an interview — one migration
file costs two minutes and signals discipline.
**Overkill when:** the full zero-downtime expand/contract choreography (add column →
dual-write → backfill → cut over → drop) matters only with live traffic. In an interview:
*mention* it, don't perform it.
**Say it live:** "Schema lives in migration files, not in my shell history — that's why
a clean clone works. With live traffic I'd do expand-and-contract; here, a forward
migration is the right size."

### B10. Concurrency & race conditions — *situational to handle, universal to identify*

**What:** two concurrent requests contending on one invariant ("last open slot").
Defense hierarchy, strongest-and-cheapest first: (1) DB **unique constraint** —
the database serializes it for you; (2) **atomic conditional write**
(`UPDATE … SET taken = taken + 1 WHERE taken < capacity`, check rows-affected);
(3) transaction with row lock (`SELECT … FOR UPDATE`); (4) app-level locks — last
resort, doesn't survive multiple instances.
**Why it works:** any check-then-act done in application code has a gap between the
check and the act; two requests both pass the check. Pushing the invariant into the
database closes the gap because the DB is the single serialization point that all
instances share.
**Applies when:** shared finite resources under concurrent writes — booking capacity,
inventory, balances. The universal part: even when you skip handling it, you must be
able to *point at the line where the race lives*.
**Overkill when:** `FOR UPDATE` on read-mostly data, pessimistic locks everywhere,
distributed-lock services for a single-Postgres system.
**Say it live:** "Two workers claiming the last slot is a textbook check-then-act race.
I won't guard it in app code — I push the invariant into Postgres with an atomic
conditional update and check rows-affected. App-level checks lose the moment there are
two server instances."

---

## MODE PLAYBOOKS

### EXTEND mode (practice round: walk in, extend the take-home)

1. **Read pass before any change** — fill the TUNING LAYER below while reading: how does
   this repo validate, handle errors, name things, test? Fifteen minutes here is what
   "knowing the codebase cold" looks like from outside.
2. **Conform over improve.** New code is indistinguishable in style from existing code.
   A better pattern introduced mid-repo is a net negative: two conventions now coexist.
3. **Don't refactor beyond the ask.** Debt you notice gets *named out loud* ("I'd
   consolidate these two validators; not doing it now") — that scores as judgment.
   Fixing it unprompted scores as scope-creep.
4. **New requirements land as new slices** following existing seams. If the seam is bad,
   say so, propose the seam you'd add, ask before adding it (A3).

### GREENFIELD mode (target round: 90 min, ambiguous, empty repo)

1. **Minute one is PLAN.md, not code** (artifact #3). Scope exceeds the clock *by
   design* — triage is the test. MVP = the thinnest end-to-end slice that answers the
   problem statement.
2. **Walking skeleton first:** repo boots, one trivial endpoint responds, test runner
   runs one trivial test. Only then, the first real slice. This front-loads all
   environment risk into minute ~10, when it's cheap.
3. **Structure ceiling:** `src/domain` (pure rules) · `src/api` (handlers) · `src/db`
   (storage + migrations) · `test/`. Nothing more until a slice demands it.
4. **Defer by default:** auth, Docker, CI, rate limiting — out of scope unless the
   problem statement says otherwise. Deferrals are listed in PLAN.md's out-of-scope,
   which converts "didn't get to it" into "chose not to."

---

## TUNING LAYER — the only per-repo section

Fill this in during the read pass (EXTEND) or as decisions are made (GREENFIELD).
Everything above this line ships unchanged to any repo.

### Repo facts
- Stack / framework / runtime version: ______
- Install: ______        Run: ______
- Test: ______           Lint / typecheck: ______
- Migrations tool + command: ______
- Seed / fixture data: ______

### Conventions observed (EXTEND) or chosen (GREENFIELD)
- Validation pattern & library: ______
- Error handling pattern (typed results? exceptions? envelope shape?): ______
- Test style & file locations: ______
- Naming / module layout conventions: ______

### Danger zones
- Files/areas not to touch: ______
- Known debt to *name*, not fix: ______
- Anything that looks broken on clean clone: ______

### Domain glossary
- ______ (terms the problem statement uses, defined precisely — in eligibility/compliance
  domains, half the ambiguity hides in vocabulary)

### Stack defaults (pre-filled per interview prep, adjust in the room)
- Greenfield default: **TypeScript** — Node 20+, Fastify (or Next.js API routes if the
  problem is UI-adjacent), Postgres, zod at boundaries, vitest.
- Python swap (if the problem favors it): FastAPI + Pydantic + pytest — Pydantic models
  ARE the B2 boundary; same contract, same playbook, only the tuning layer changes.
