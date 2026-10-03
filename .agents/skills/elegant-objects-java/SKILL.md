---
name: elegant-objects-java
description: >-
  Write, review, test and maintain Java projects in Yegor Bugayenko's Elegant Objects (EO) style:
  immutable objects, code-free constructors, no null / getters / setters / statics / instanceof /
  implementation inheritance — reuse by decorators. Includes Maven + Qulice quality gates,
  JUnit 5 + Hamcrest one-assertion tests, CI/CD (GitHub Actions + Rultor), and repository,
  documentation and release conventions. Use when writing, refactoring, reviewing or scaffolding
  Java code; setting up a Java build, CI/CD or release; writing Java tests; or applying EO
  principles to an existing codebase.
license: MIT
---

# Elegant Objects Java

A complete working skill for owning a Java codebase the Elegant Objects way: design, tests,
build, CI/CD, release and documentation. Everything here is enforceable by the build, not just
advice.

## When to use

- Writing or refactoring Java (new classes, methods, modules).
- Reviewing a Java pull request or auditing a Java repository.
- Setting up a Maven build, Qulice static analysis, coverage/mutation gates.
- Writing JUnit tests (especially EO-style one-assertion tests).
- Setting up CI/CD, a release pipeline, or repository/documentation conventions.
- Scaffolding a new Java library or service.

**When not to use:** non-Java codebases (use the same ideas, but the tooling here is JVM-specific);
throwaway scripts; projects that explicitly forbid these conventions.

## The 30-second version

1. A class is where objects are **born**; an object is a **representative of a real entity**
   (a file, a page, a user), not a data bag and not a bag of functions.
2. Make it `final`, fields `private final`, no setters — **immutable**.
3. Constructors **only assign** (no code, no `new` of collaborators, one primary ctor declared last).
4. Implement **interfaces**; add behavior by **decorating** (`implements X` wrapping another `X`),
   never by `extends` of a concrete class.
5. No `null`, no `static`, no getters/setters, no `instanceof`/casting, no `-er` names.
6. Tests: build the object, then **one** `assertThat(...)`; no fixtures, prefer fakes to mocks.
7. The build **fails** on any Qulice violation and on coverage below the gate.
8. Every change traces to a ticket; every bug fix ships a reproducing test.

## The canon (memorize the shape)

Eleven principles (full detail in `references/01-design.md`):

| # | Principle | Instead |
|---|-----------|---------|
| 1 | No `null` | Null Object, empty collection, or throw |
| 2 | No code in constructors | assign now, compute in methods (lazy) |
| 3 | No getters/setters | tell, don't ask; expose behavior |
| 4 | No mutable objects | `private final`; return a new object |
| 5 | No `-er` names | name what it *is* (`FileReader` → `DataFile`) |
| 6 | No static methods (not even private) | an object you inject |
| 7 | No `instanceof`/casting/reflection | polymorphism + decorators |
| 8 | No public method without an interface | every public method overrides an interface |
| 9 | No statements in a test but `assertThat` | arrange by composing objects |
| 10 | No ORM / ActiveRecord | wrap persistence behind objects |
| 11 | No implementation inheritance | `final` class + decoration |

Seven virtues of a good object: exists in real life · works by contracts (interfaces) · is unique
(encapsulates something) · is immutable · has nothing static · name is not a job title · is
`final` or `abstract`.

## Non-negotiable rules (the build must enforce these)

1. **No `null`** in any public contract — never accept, return or signal with `null`.
2. **No `public static`** — `main`, JUnit providers and `private static final` constants only.
3. **No utility classes, no singletons** — inject via constructors.
4. **Classes are `final` or `abstract`** (envelope bases may be non-final with `final` methods).
5. **All fields `private final`**; objects immutable; mutators return a new object.
6. **Constructors only assign** or delegate `this(...)`; no parsing, validation, I/O, `new` of peers.
7. **Exactly one primary constructor, declared last**; secondaries funnel via `this(...)`.
8. **`new` only in secondary constructors.**
9. **No `get`/`set`** prefixes; expose behavior.
10. **Every public method implements an interface**; keep interfaces short (≤3 methods).
11. **No implementation inheritance** — composition/decorators only.
12. **No `-er` job-title names.**
13. **No `instanceof`, casting or reflection** (except `equals(Object)`).
14. **Fail fast with checked exceptions**; never swallow, never catch-and-log; recover once at the top.
15. **One `assertThat` per test**; no `@Before`, no shared fields; fakes over mocks.
16. **Qulice is mandatory** and fails the build.
17. **Coverage/mutation gates are in the POM** and fail the build.
18. **Every bug fix ships a reproducing test.**
19. **One-command, tag-driven, automated release**; read-only `master`, PR-only, bot-merged.
20. **Every change has a ticket and every commit references it**; never rewrite history.

## How to work on any task

Follow this loop (detail in `references/09-recipes.md` and `references/07-process.md`):

```
1. Understand the ticket. If there is none, create one (what + minimal reproduction).
2. Reproduce in a failing test (or a @Disabled test + @todo puzzle if you cannot yet).
3. Design by naming real entities: which objects exist, which interfaces they implement,
   which decorator adds the new behavior. Prefer more, smaller classes.
4. Implement: final classes, private final fields, code-free ctors, no null/static/instanceof.
5. Verify locally:  mvn --errors --batch-mode clean install -Pqulice
6. Commit: branch `<ticket>` off `master` (never commit to trunk); message starts with #<ticket>;
   atomic commits, one small concern per PR; merged by the bot, not by hand.
7. Release (if asked): tag semver -> versions:set -> signed deploy (see references/05-cicd-release.md).
```

Rules of engagement for an agent:

- **Never invent scope.** Do exactly the ticket; file new findings as puzzles/tickets.
- **Never bypass the gate.** Do not add blanket `@SuppressWarnings`; a narrow exclusion needs a
  ticket and a comment.
- **Never rewrite history** (no force-push, no deleted comments).
- **Never merge into a broken `master`**; a build fix is its own PR.
- **Leave the repo greener**: a fix + its test + a reported bug/puzzle.
- **Close the review loop**: fix nits in the same PR; turn anything larger into a linked follow-up
  issue and answer every comment — never silently ignore one.

## Definition of Done

- [ ] Branch `<ticket>` off read-only `master`; one small PR tied to a ticket; atomic commits,
  message starts with `#<ticket>`; merged by Rultor (no squash, no hand-merge).
- [ ] Behavior covered by tests: one statement per test, Hamcrest `assertThat`.
- [ ] Any bug has a test that reproduces it first.
- [ ] `mvn --errors --batch-mode clean install -Pqulice` is green.
- [ ] Coverage/mutation above the gate; no new static/null/`instanceof` in business logic.
- [ ] Public API has interfaces; classes `final`; fields `private final`.
- [ ] External interfaces documented (README/usage); no inline narration.
- [ ] No secrets in history; SPDX header on new files; license unchanged/consistent.
- [ ] Every review comment addressed: nits fixed in this PR, larger changes filed as a linked issue.

## Reference map (read only what the task needs)

| Topic | File |
|---|---|
| OOP design, constructors, immutability, names, decoration, exceptions | `references/01-design.md` |
| Testing: TDD/BDD, one-assert, fakes, coverage, integration | `references/02-testing.md` |
| Code style + Qulice rules + quality gate recipe | `references/03-code-style.md` |
| Maven: parent POM, dependencies, profiles, reproducible builds | `references/04-build-maven.md` |
| CI/CD: GitHub Actions, Rultor, secrets, releases, deploy | `references/05-cicd-release.md` |
| Repository & docs: SPDX, README, gitignore, traces, PDD | `references/06-repository-docs.md` |
| Process: tickets, DoD, reviews, roles, AI-agent etiquette | `references/07-process.md` |
| Tooling catalogue: build, test, runtime, quality | `references/08-tooling.md` |
| Step-by-step recipes (scaffold, feature, bug, refactor, gate, release, review) | `references/09-recipes.md` |
| Branch names, commit format/frequency, merge & Rultor policy | `references/10-branching-and-commits.md` |
| Reviewing a PR: checklist, report shape, label approval | `references/11-reviewer-playbook.md` |
| Full knowledge base (book + 300+ articles + 10 real repos) | `knowledge/elegant-objects-java.md` |
| Distilled 315-rule checklist with priorities | `knowledge/distilled-rules.md` |

Deep-dive only when needed — the knowledge base is large. Start from the reference file for the
task and search it; open `knowledge/` for provenance and edge cases.

## The build command

```bash
mvn --errors --batch-mode clean install -Pqulice
```

Libraries also gate coverage (Jacoco) and mutation (PIT); public APIs gate compatibility (Revapi).

## Anti-patterns at a glance (do not write these)

`null` returns/params · utility classes & `public static` · mutable objects & setters · getters ·
DTOs & naked data · ORM/ActiveRecord · singletons · `-er`/`-or` names · `instanceof`/casts/reflection ·
implementation inheritance · builders & fluent mutable chains · DI containers · behavior-carrying
annotations · `Optional` as null-replacement · catch-and-log · comments narrating code ·
God objects · `@Before` fixtures in tests · assertions on internals.

## Pragmatic deviations (allowed, but must be written down)

Real EO repositories are gate-enforced pragmatists, not purists. Tolerated, narrowly:

- `private static final` constants; `public static void main` launchers.
- `Lombok @EqualsAndHashCode`/`@ToString` on immutable value objects (`provided` scope).
- Javadoc on public API (Qulice requires it) even though inline narration is banned.
- JDK/third-party `null` guards wrapped inside a decorator (`Opt`, `NotNull`).
- Legacy code gets an explicit, ticket-linked `@SuppressWarnings`/`@checkstyle` — never a blanket one.

When you deviate, say why in a comment or the PR description. Never silently.

## Enforcing this skill (for humans)

A rule that isn't enforced gets ignored. Wire the [MUST] rules into the build:

- Qulice in CI (`-Pqulice`) fails on style/design/EO violations.
- Jacoco + PIT fail below the coverage/mutation thresholds.
- Revapi fails on un-justified public API breaks.
- The merge is done by a bot on a read-only `master` (Rultor), so the gate cannot be bypassed.

See `references/03-code-style.md` and `references/05-cicd-release.md` for ready-to-paste config.
