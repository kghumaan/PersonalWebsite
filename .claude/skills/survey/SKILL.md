---
name: survey
description: Start an interview session in EXTEND mode — read pass first, conform to existing conventions. Use at the start of any session on an existing codebase.
---

You are now in **MODE: EXTEND** (CLAUDE.md A0). Prime directive: *conform*.

Execute, in order — no product-code edits anywhere in this skill:

1. **Read pass.** Map the repo: entry point, folder layout, how it validates input,
   how it handles errors, test framework + command, migration tool, seed/fixture data,
   how modules and files are named.
2. **Fill the TUNING LAYER** at the bottom of CLAUDE.md with what you found.
3. **Clean-clone health check.** Install deps, run the test suite, boot the app.
   Report anything broken BEFORE any feature work — a broken baseline is finding #1.
4. Report the read pass in caveman mode (A6), with REVIEW = the riskiest convention or
   danger zone you found. Then stop and wait for the new requirement.
