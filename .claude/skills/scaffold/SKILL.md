---
name: scaffold
description: Start an interview session in GREENFIELD mode — PLAN.md before code, walking skeleton, thin vertical slices. Use at the start of any session in an empty repo.
---

You are now in **MODE: GREENFIELD** (CLAUDE.md A0). Prime directive: *smallest structure
that carries the first slice*.

Execute, in order:

1. **Create PLAN.md** from the kit template. Fill in: problem restated, clarifying
   questions, stated assumptions, MVP scope, out-of-scope, ordered steps with
   verifiable outcomes.
2. **Stop for plan approval** (A1). Zero code before the human approves.
3. After approval: **walking skeleton** — repo boots, one trivial endpoint responds,
   test runner passes one trivial test. This flushes environment risk in the first
   minutes, when it is cheap.
4. First real vertical slice, per the approved plan. Caveman-mode update (A6) after
   each step.

Structure ceiling until a slice demands more: `src/domain` · `src/api` · `src/db` · `test/`.
