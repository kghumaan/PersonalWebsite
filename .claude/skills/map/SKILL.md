---
name: map
description: Kickoff read fan-out — launch parallel read-only scout agents on distinct angles, synthesize their reports into one map with conflicts surfaced. Use at minute ~2 of any session, both modes; pairs with PLAN.md.
---

Launch **parallel read-only subagents** (the `scout` agent type), one per angle, then
synthesize. You remain the only writer; scouts never edit.

Pick angles by mode (3 scouts default; 4 if the repo/problem is large):

**EXTEND** (existing codebase):
1. API surface — routes, handlers, status codes, response shapes, validation entry points.
2. Domain rules — every business rule/branch, where implemented, which have tests.
3. Storage — schema, migrations, how queries are built, any check-then-act writes.
4. (optional) Tooling health — install/run/test/lint commands actually work.

**GREENFIELD** (problem statement, no code):
1. Domain — entities, vocabulary to pin down, edge cases hiding in the problem statement.
2. Data model — candidate schema, the invariants it must protect, migration plan.
3. Risk — the slice most likely to blow up (messy input? race? reconciliation?) and the
   cheapest way to de-risk it early.

Then synthesize into ONE report:
- Merged map (deduplicated, scouts' file:line refs preserved)
- **CONFLICTS** — anywhere scouts disagree or a finding is uncertain (this section is
  the reason synthesis exists; never omit it)
- Feed conventions/danger zones into the CLAUDE.md TUNING LAYER, and open questions
  into PLAN.md's clarifying-questions section if it exists.

Close with a caveman block (A6): REVIEW = the single most consequential conflict or risk.

Why this shape (say it live): reads parallelize safely — scouts burn their own context
and return summaries, keeping the build session's context clean; writes stay serialized
per A9 because the human can only review one diff stream.
