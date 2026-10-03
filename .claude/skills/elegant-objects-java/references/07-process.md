# 07 — Working Process

Read this when you pick up a task, plan work, report progress, or open a PR.
Every rule carries a priority tag: **[MUST]** blocks the task, **[SHOULD]** is the strong default
(deviate only with a written reason in a ticket), **[NICE]** is guidance.

Source: `knowledge/distilled-rules.md` §13 (Project & process), §12, §7, §9, §11.

---

## 1. Ticket-first

- **[MUST]** Every ticket is a **complaint with a minimal reproduction**. State the observed bad
  behavior and the smallest sequence that reproduces it. No reproduction, no work.
- **[MUST]** No "BTW" scope creep. If you notice something else, file a separate ticket; do not fix
  it in this one.
- **[MUST]** Every idea starts with a ticket, and every change ties to a ticket and a same-named
  branch.
- **[MUST]** Communicate through tickets — not meetings, chat, or email. The task author is the
  single point of contact.
- **[SHOULD]** Let the reporter close the ticket; the author does not self-close.
- **[NICE]** Automate ticket intake with puzzles and 0pdd rather than a manager.

### Issue template

```markdown
## What is wrong

<one paragraph — the observed behavior, not the suspected cause>

## Minimal reproduction

1. <step>
2. <step>
3. <step>

Expected: <what should happen>
Actual:   <what happens>

## Environment

- Version:
- JDK / OS:
- Build command: `mvn --errors --batch-mode clean install -Pqulice`
```

---

## 2. Start with a failing test

- **[MUST]** Begin every task with a **failing test that reproduces the problem**. The test is the
  ticket made executable.
- **[MUST]** Every bug fix ships with a reproducing test. No fix without it.
- **[SHOULD]** If you run out of budget, commit the `@Disabled` test plus a `@todo` puzzle and close
  the ticket, describing what remains.
- **[MUST]** One assertion per test; construct the subject and then a single
  `MatcherAssert.assertThat(...)`.
- **[SHOULD]** Prefer bug-driven development (reproduce, then fix) over pure test-first.
- **[NICE]** Submit the test separately from the fix to prevent "adjust the tests to pass".

---

## 3. Fix-and-break discipline

- **[MUST]** A deliverable is never just a fix. It is **fix + tests + a reported bug or a puzzle**.
- **[SHOULD]** Make bugs welcome and pay for discovering them; one bug per 2–3 completed tasks is
  healthy.
- **[SHOULD]** Write a `@todo` puzzle when a fix reveals adjacent work you cannot do now.
- **[MUST]** Never leave a bare `TODO`/`FIXME`; use the puzzle form (`@todo #N:30min ...`).
- **[SHOULD]** Let programmers chase speed; the pipeline enforces quality.

---

## 4. Definition of Done

A change is Done only when **all** of these hold:

```text
[ ] Tied to a GitHub issue; commit subject starts with #<issue>
[ ] A test that failed before the change and passes now
[ ] All tests pass (one assertion each, no fixtures, no @Before)
[ ] Qulice is green: mvn --errors --batch-mode clean install -Pqulice
[ ] Line coverage and mutation above the POM gate
[ ] Fakes updated with the interface (the test did not change)
[ ] Public Javadoc added for new external types/methods
[ ] README/docs/version synced if behavior or usage changed
[ ] One small PR; no unrelated changes; no history rewrite
```

- **[MUST]** Done means the **author's acceptance** — pay for closed deliverables, never for hours.
- **[MUST]** Never merge into a broken master. A build fix is its own PR.
- **[SHOULD]** Report artifacts (what exists), not effort (hours spent).
- **[SHOULD]** Keep a plan with an owner and a reviewer per artifact.

---

## 5. Micro-tasks with a fixed budget

- **[SHOULD]** Decompose into 30–60-minute micro-tasks with a fixed budget.
- **[SHOULD]** When a micro-task overflows its budget, stop and convert the remainder into a PDD
  puzzle; do not silently expand scope.
- **[SHOULD]** Decompose top-down by **user value**, not by technical layers.
- **[MUST]** Keep functional and non-functional requirements separate and measurable.
- **[NICE]** Estimate cost as price-per-unit (hits-of-code / quality), never a fixed total.
- **[NICE]** Track Hits-of-Code, not lines of code.

---

## 6. Four project phases

- **[SHOULD]** Run in four phases:
  1. **Thinking** — write the specification; no code.
  2. **Building** — one accountable architect builds a prototype.
  3. **Fixing** — mass bug work; stabilize the prototype.
  4. **Using** — bug fixes only; the product is in service.
- **[MUST]** Keep the phase visible; do not do "Building" work during "Using".
- **[SHOULD]** In Thinking, write a short **Product Vision** in four sections: actors, features per
  actor, quality requirements, and constraints.
- **[SHOULD]** Cap features per actor and cap quality requirements explicitly.

### Product Vision sketch

```markdown
1. Actors: <who uses the product>
2. Features per actor: <≤N features each>
3. Quality requirements: <measurable, non-functional>
4. Constraints: <budget, stack, deadlines>
```

---

## 7. Roles & reviews

- **[SHOULD]** Appoint exactly **one accountable architect**. Decisions are documented artifacts,
  not hallway agreements.
- **[SHOULD]** Assign personal responsibility; rules drive performance decisions, not mood.
- **[NICE]** Run independent reviews; pay per bug found; ask for criticism; rotate reviewers.
- **[SHOULD]** A reviewer must **prove the code is bad**; the author need not prove it good.
- **[NICE]** Resolve review conflicts exactly three ways — accept, hold firm, or appeal to the
  architect — never compromise.
- **[MUST]** Trust without control becomes chaos: pair delegation with tests, gates, and rules.
- **[NICE]** Prefer rules and planning over an unpredictable manager.

### Bug-bounty mindset

- **[SHOULD]** Treat every bug as a paid discovery; nothing but a bug report or a PR counts as a
  contribution.
- **[SHOULD]** Require bug reports or PRs from contributors; other feedback is welcome but does not
  count.
- **[MUST]** Blame the project for unclear code, not yourself; file a docs/source bug ticket instead
  of working around it.

---

## 8. Traceability & estimation

- **[MUST]** Every change ties to an issue and a same-named branch; commits start with `#123`.
- **[MUST]** Never rewrite history: no force-push, no deleted commits or comments.
- **[SHOULD]** Let an automated merge bot merge and report in the PR.
- **[NICE]** Estimate by price-per-unit (hits-of-code / quality), never a fixed total.
- **[SHOULD]** Report artifacts with an owner and a reviewer; keep the list current.

---

## 9. How an AI agent behaves in this process

- **[MUST]** One concern per PR. Never bundle a fix, a refactor, and a docs change.
- **[MUST]** Ask questions via a ticket comment, not by guessing. If the ticket is ambiguous, request
  clarification in the ticket and stop.
- **[MUST]** Never rewrite history — no force-push, no amended commits on a shared branch.
- **[MUST]** Always leave a reproducing test behind for any bug you fix.
- **[MUST]** Never merge into a broken master; never push a red build.
- **[SHOULD]** Start from the failing test; if you cannot reproduce, add a passing test proving the
  intended behavior and say so in the ticket.
- **[SHOULD]** Keep each PR small (<50 hits-of-code for newcomers) and single-purpose.
- **[SHOULD]** Do mechanical chores (rewording issues, small refactor PRs, docs sync) as separate
  small concerns — one per PR.
- **[MUST]** Cite the rules you followed; when you deviate from a **[SHOULD]**, state the reason in
  the ticket.
- **[SHOULD]** Report progress as closed deliverables, not as time spent.

### Task workflow (numbered checklist)

```text
1. Read the ticket; confirm it has a minimal reproduction. If not, ask in the ticket and stop.
2. Pick the branch named after the issue; confirm the base is green.
3. Write a failing test that reproduces the problem.
4. Run the build and confirm the test fails for the expected reason.
5. Make the smallest change that makes the test pass.
6. Add tests for any new behavior; keep one assertion per test.
7. Run the full build: mvn --errors --batch-mode clean install -Pqulice.
8. Verify the Definition-of-Done checklist.
9. If work remains out of scope, add a @todo puzzle (or file a new ticket).
10. Commit with a #<issue> subject; open one small PR.
11. Address every review comment: fix nits in this PR, file a follow-up issue for anything larger; do not force-push.
12. Let the merge bot merge; let the reporter close the ticket.
```

---

## 10. PR template

```markdown
## What

<one paragraph: the change and the issue it closes>

Closes #<issue>

## Why

<the complaint from the ticket, in one sentence>

## How

- <key decision>
- <rejected alternative, if any>

## Checklist

- [ ] A test failed before this change and passes now
- [ ] `mvn --errors --batch-mode clean install -Pqulice` is green
- [ ] Coverage/mutation above the gate
- [ ] One concern only; no unrelated changes
- [ ] Docs/version synced if needed
- [ ] No history rewrite
```

- **[MUST]** The PR body names the issue (`Closes #N`) and states the single concern.
- **[SHOULD]** Keep the description factual; reviewers prove defects, authors state intent.
- **[SHOULD]** Do not merge your own PR; the bot merges after review.

---

## 11. Process anti-patterns (do not do)

- **[MUST]** No untraceable changes (a commit without an issue).
- **[MUST]** No volunteer static analysis — gates are mandatory in CI.
- **[MUST]** No nonstop development without a release; prefer several small releases a day.
- **[MUST]** No undocumented interfaces; document the external contract.
- **[MUST]** No ad-hoc releases; releases are tag-driven and scripted.
- **[MUST]** No unknown coverage; the gate is in the POM and binding.
- **[SHOULD]** No compromise on review conflicts — accept, hold firm, or appeal.

---

## 12. Review comments & approval

Triage every review comment — never silently ignore it:

- **[MUST] Fix nits in the same PR.** A tiny fix (wording, a name, a missing/incorrect test, a
  one-liner, formatting) is an extra commit on the same branch, referencing the same ticket.
  Do NOT open a new issue for it.
- **[MUST] Escalate anything larger to its own issue.** A design change, new behavior, a big
  refactor, or anything outside the PR's scope becomes a separate issue; link it from the PR
  (`Follow-up: #NNN`) and keep this PR focused. Do NOT grow the PR.
- **[MUST] Answer every comment.** Reply (`@reviewer`) with either the fix commit or the
  follow-up issue link and the reasoning. No unexplained silence.
- **[MUST] The reviewer closes the loop.** After the author pushes, the reviewer re-reviews,
  **resolves the threads it opened** once addressed, and records the verdict.

### Approval in a single-account setup (labels)

GitHub forbids approving your own PR. When the author and the reviewer share one account, labels
are the approval signal:

- `needs-review` — a PR awaiting review.
- `approved` — the reviewer's verdict: all findings addressed, nits resolved.
- `reviewed` — a review was posted (optional marker).

The reviewer applies/removes these labels and resolves the threads it opened. Final merge stays
with the maintainer.
