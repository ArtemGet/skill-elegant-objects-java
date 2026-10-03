# 11 — Reviewer playbook

How an independent reviewer (a human, or an agent subagent) works a pull request. Pair with
`references/07-process.md` §12 (review comments & approval).

## Before reviewing

- Read `SKILL.md` and the reference for the changed area (design / testing / style / CI).
- Re-read the linked issue: the PR must close EXACTLY that issue — no scope creep.

## What to check

- **Correctness** — the change does what the issue asks; edge cases handled.
- **Tests** — a test reproduces the bug (fails before), one assertion per test, Hamcrest matchers,
  `final` test class, no fields, named `*Test`.
- **Style / EO** — `final` classes, `private final` fields, no statics / `null` / `instanceof`
  (except inside `equals`), no blind casts.
- **Process** — branch names the issue, commits reference `#<issue>`, one concern, no history rewrite.
- **Gates** — Qulice + jtcop + coverage green; CI runs visible.

## Checking CI (do not hang the session)

- Read the checks via the GitHub MCP (`pull_request_read` → `get_check_runs`). Never call the GitHub
  API with a raw token — MCP only.
- If a required check is **red** and its log is not reachable through MCP, record the failing check
  name + job URL and STOP to report it — do **not** try to reproduce the full CI locally.
- Never run a foreground build (or `docker run`/`docker pull` of a large image): it blocks with no
  output and looks hung. If a local check is truly unavoidable, run it in the background with a log
  file and a timeout, and poll the tail in short increments (see `references/04-build-maven.md` §14).

## How to report

- Post a review; add line comments for anything specific.
- Triage every comment (see §12): a **nit** → the author fixes it in this PR; anything **larger** →
  a linked follow-up issue. The reviewer decides the category, not the author.
- **Do NOT editorialize about platform limits.** Never write things like "submitted as COMMENT —
  GitHub blocks approving one's own PR", "can't approve my own PR", or similar meta-commentary —
  not in the PR and not in the report back to the orchestrator. In a single-account setup the
  **label is the approval**; apply it silently.
- Keep the review terse and factual: verdict, evidence (`path:line` / commit), requested changes.

## Close the loop

- After the author pushes, re-review the new commit(s).
- Reply to each of your comments ("Addressed in `<sha>`").
- **Resolve** the review threads you opened.
- Apply `approved` (and remove `needs-review`) once nothing is outstanding.
- Final merge is the maintainer's; **do not merge**.

## Output shape (back to the orchestrator)

```text
verdict: APPROVED | CHANGES_REQUESTED
checks:  <bullets: what was verified>
actions: <thread ids resolved; label applied>
remaining: none | <concern>
```

No commentary about self-approval or platform restrictions. If a formal `Approve` is unavailable,
the label carries the verdict — say nothing else about it.
