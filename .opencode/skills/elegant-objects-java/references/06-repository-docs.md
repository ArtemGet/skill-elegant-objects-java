# 06 — Repository & Documentation Conventions

Read this when you create a repository, add a file, write a README, or touch licensing.
Every rule carries a priority tag: **[MUST]** fails the build / blocks the PR, **[SHOULD]** is the
strong default (deviate only with a written reason in a ticket), **[NICE]** is review guidance.

Source: `knowledge/distilled-rules.md` §12 (Repository & docs conventions), §13, §11, §10.

---

## 1. Licensing — one license, everywhere

- **[MUST]** Put an SPDX header on **every** source and config file (`.java`, `.xml`, `.yml`, `.md`,
  `.sh`, `.properties`). No exceptions, including test code and generated-adjacent files.
- **[MUST]** State **one** license identically in all five places: the SPDX header, `LICENSE.txt`,
  `pom.xml` (`<licenses>`), `README` badge, and `REUSE`/`REUSE.toml`. A mismatch is a bug — requs
  shipping a BSD POM with an MIT SPDX header is the counter-example.
- **[MUST]** Enforce it automatically: `license-maven-plugin` at the `verify` phase, plus the REUSE
  tooling and a `reuse` CI workflow. Header drift must fail the build, not a review.
- **[MUST]** Map non-source globs (images, JSON fixtures, lock files) in `REUSE.toml`; do not leave
  them unmapped.

### SPDX header snippet (Java)

```java
/*
 * SPDX-FileCopyrightText: Copyright (c) 2024 Your Name
 * SPDX-License-Identifier: MIT
 */
package com.example;
```

### SPDX header snippet (YAML / config)

```yaml
# SPDX-FileCopyrightText: Copyright (c) 2024 Your Name
# SPDX-License-Identifier: MIT
```

- **[MUST]** Keep the copyright line and the license identifier in this order; keep the header
  identical across files of the same project.
- **[SHOULD]** Prefer a permissive license (MIT/BSD-3) for libraries unless the project already
  commits to another.

### `REUSE.toml` sketch

```toml
version = 1
[[annotations]]
path = ["docs/**", "*.json", "*.png"]
SPDX-FileCopyrightText = "Copyright (c) 2024 Your Name"
SPDX-License-Identifier = "MIT"
```

---

## 2. `.gitattributes` and `.gitignore`

- **[MUST]** Normalize line endings with `.gitattributes`: `* text=auto eol=lf`.
- **[MUST]** Declare ident substitution for the file types that use an ident line:
  `*.java ident` and `*.xml ident`.
- **[MUST]** Keep `.gitignore` small and tool-focused. Include `.claude/`, IDE files
  (`.idea/`, `*.iml`, `.vscode/`), build output (`target/`), and `node_modules/`.

```gitattributes
* text=auto eol=lf
*.java ident
*.xml ident
*.bat text eol=crlf
```

```gitignore
# build output
target/
build/
out/
*.class
*.jar

# logs & run artifacts
*.log
*.exec
*.tmp
hs_err_pid*
replay_pid*

# tooling / IDE / OS
node_modules/
.claude/
.idea/
*.iml
.vscode/
.DS_Store

# secrets
.env
*.pem
*.key
```

- **[SHOULD]** Do not ignore files that are part of the build contract (wrapper, `.mvn/`).
- **[MUST]** Never commit logs, coverage dumps, build output, IDE/OS files, downloaded sources,
  or run artifacts — they are noise and can leak data. Add them to `.gitignore` **before** the
  first commit; if one slips in, remove it in a dedicated commit (never by rewriting history).
- **[MUST]** Never commit secrets, tokens, API keys, `.env`, or local machine paths; keep
  credentials in Maven profiles or environment variables, never in the POM or a committed config.
  A token that leaks into history must be rotated immediately.

---

## 3. Package documentation

- **[MUST]** Add a `package-info.java` with a one-line Javadoc to **every** package.
- **[MUST]** End the sentence with a period; Qulice enforces the format.

```java
/*
 * SPDX-FileCopyrightText: Copyright (c) 2024 Your Name
 * SPDX-License-Identifier: MIT
 */

/**
 * Core immutable value objects.
 * @since 0.1.0
 */
package com.example.core;
```

- **[SHOULD]** Document the package's role, not a list of its classes; the classes document themselves.
- **[SHOULD]** Add `@since` to every package, matching the release that introduced it.

---

## 4. The elegant README (external contract)

The README **is** the project's external contract. Treat it as production code: it is versioned
Markdown, reviewed via PRs, and read by strangers first.

- **[MUST]** Lead with the **problem statement in the first paragraph**. Say what pain the project
  removes before naming the project.
- **[MUST]** Put a badges row immediately after the title: EO principles, Rultor/bot, CI status,
  coverage, license.
- **[MUST]** Include an install snippet (copy-pasteable, exact coordinates and version).
- **[MUST]** Include a usage cookbook — short, compiling examples that show the primary flow.
- **[MUST]** Include an Architecture section: the top-level objects, their contracts, and how they
  compose.
- **[MUST]** Include a "How to contribute" section with the **exact** command:
  `mvn --errors --batch-mode clean install -Pqulice`.
- **[SHOULD]** Keep README lines ≤80 columns.
- **[SHOULD]** Use second-level headers (`##`) only; keep one blank line between blocks.
- **[SHOULD]** Keep the opening free of license/changelog/contributor boilerplate — those go lower
  or into their own files.
- **[SHOULD]** Keep the build command in the README identical to the CI command; drift is a bug.

### README template

```markdown
# Project Name

[![EO principles respected](https://img.shields.io/badge/EO-1215-green.svg)](https://www.elegantobjects.org)
[![License](https://img.shields.io/badge/license-MIT-green.svg)](LICENSE.txt)
[![Build](https://github.com/OWNER/REPO/actions/workflows/build.yml/badge.svg)](https://github.com/OWNER/REPO/actions)
[![Coverage](https://img.shields.io/badge/coverage-80%25-green.svg)](#)

Problem: <one paragraph — the concrete pain this project removes>.

## Install

```xml
<dependency>
  <groupId>com.example</groupId>
  <artifactId>example</artifactId>
  <version>0.1.0</version>
</dependency>
```

## Usage

```java
final Thing thing = new SmartThing(new DefaultThing("input"));
System.out.println(thing.result());
```

## Architecture

- `Thing` — the public contract.
- `DefaultThing` — the primary implementation.
- `SmartThing` — convenience decorator.

Objects compose; there are no static utilities and no mutable state.

## How to contribute

Fork, create a branch named after the issue, and run the full build before opening a PR:

```bash
mvn --errors --batch-mode clean install -Pqulice
```

Every change needs a GitHub issue; commit messages start with `#123`.
```

- **[SHOULD]** Keep badges real: a badge that points nowhere, or a coverage badge without a target,
  is worse than no badge.
- **[NICE]** Add one screenshot or output sample for anything visual or CLI-shaped.

---

## 5. Docs as versioned Markdown

- **[MUST]** Keep documentation as versioned Markdown in the repository; change it through the same
  PR flow as code. Docs out of sync with the code are a defect.
- **[SHOULD]** Document **external interfaces** only; working software beats internal narration.
- **[SHOULD]** Record key decisions as short records: the decision, the rejected alternatives, and
  the assumptions/risks/concerns. Keep them next to the code or in `docs/`.
- **[NICE]** Add `CITATION.cff` when the project produces research output, so it can be cited.
- **[NICE]** Keep a `CHANGELOG.md` or generate release notes from tags — never let changes go
  unreported.
- **[SHOULD]** Never duplicate facts across README and `docs/`; link instead of copying.

### `CITATION.cff` sketch

```yaml
cff-version: 1.2.0
title: Project Name
message: If you use this software, please cite it as below.
type: software
authors:
  - family-names: Doe
    given-names: Jane
license: MIT
version: 0.1.0
```

---

## 6. Traceability

- **[MUST]** Every change has a GitHub issue. No issue, no change.
- **[MUST]** Commits start with `#123` (the issue number) so the tracker auto-links:
  `#123 Add NullThing`.
- **[MUST]** Never rewrite history: no force-push, no deleted commits, no deleted review comments.
- **[MUST]** Branch per issue off read-only `master`, named after the issue (`42`, or `42-topic`);
  never commit to trunk. The PR is merged by the bot (a true merge commit, no squash) — see
  `10-branching-and-commits.md` for the naming, commit and merge policy.
- **[SHOULD]** Keep one change = one small PR: one logical change per commit, no mixing
  fix/refactor/format. Aim for 40–100 hits-of-code (CI fails above 200 changed lines).
- **[SHOULD]** Address review comments by `@nickname`; let the reporter close the ticket.

### Commit-message convention

```text
#123 Add NullThing

Explain in one short paragraph why the change is needed and what it
does. Reference the problem, not the mechanics ("no null", not "added
a class").

Refs #123
```

- **[MUST]** Keep the first line imperative and ≤72 columns.
- **[MUST]** Reference the issue in the subject **and** in the body.
- **[SHOULD]** One logical change per commit; do not mix a fix with a refactor.

---

## 7. PDD puzzles (`@todo`)

- **[MUST]** Use PDD puzzles instead of bare `TODO`/`FIXME`. A puzzle has an issue, an estimate, and
  a description.
- **[SHOULD]** Let 0pdd turn puzzles into GitHub issues automatically.

### Puzzle example

```java
// @todo #123:30min Replace this null guard with a NullThing decorator once
//  the interface exposes content() on the empty entity. Tracked for the
//  next micro-task; do not block the current fix on it.
```

- **[MUST]** Format: `@todo #N:30min <description>` — issue number, estimate, then the task.
- **[SHOULD]** Keep the estimate in minutes and realistic; a puzzle is a promise, not a wish.
- **[SHOULD]** When a puzzle's work is done, remove the puzzle in the same PR that resolves it.

---

## 8. Decision records

- **[SHOULD]** Capture each significant decision as a short record with: context, decision,
  rejected alternatives, and the consequences.
- **[SHOULD]** List assumptions, risks, and open concerns explicitly rather than hiding them in
  prose.
- **[MUST]** When a tool or rule contradicts another, disable exactly one with a written rationale
  in a ticket; never blanket-suppress.

---

## 9. Build files & release metadata (repo-facing)

- **[MUST]** Keep the version `-SNAPSHOT` in source; the release bot injects the tag via
  `mvn versions:set -DnewVersion=${tag}`.
- **[SHOULD]** Centralize dependency versions in `dependencyManagement`/a BOM; children omit
  versions.
- **[SHOULD]** Keep one non-interactive build command in CI and the README; never diverge.
- **[NICE]** Commit `.mvn/jvm.config` and a Maven wrapper for reproducible builds.

---

## 10. Repo checklist (run before you open a PR)

```text
[ ] SPDX header on every new file
[ ] License identical in POM / SPDX / REUSE / README / LICENSE.txt
[ ] .gitattributes has * text=auto eol=lf
[ ] package-info.java added for each new package
[ ] README problem statement, badges, install, usage, architecture, contribute
[ ] Commit subject starts with #<issue>
[ ] No bare TODO/FIXME — PDD puzzle instead
[ ] No force-push, no history rewrite
[ ] mvn --errors --batch-mode clean install -Pqulice is green
```
