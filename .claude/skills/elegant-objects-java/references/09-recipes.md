# 09 — Recipes: end-to-end, copy-adaptable

Audience: an AI coding agent executing an EO Java task end-to-end.
Each recipe states **goal → steps → key snippets → definition of done (DoD)**.
Tags: **[MUST]** non-negotiable · **[SHOULD]** strong default · **[NICE]** case-by-case.
Canonical command throughout: `mvn --errors --batch-mode clean install -Pqulice`.

Global rules for every recipe: [MUST] every change ties to a ticket and every commit starts with
`#<ticket>`; [MUST] one small single-purpose PR, never a fix mixed with a refactor; [MUST] tests are
first-class (one `assertThat(reason, actual, matcher)` per test, no fixtures); [MUST] never rewrite
history (no force-push, no deleted commits/comments).

---

## Recipe 1 — Scaffold a new EO Java library

**Goal:** a buildable, linted, tested, licensable repo inheriting `com.jcabi:parent` that passes
`mvn … -Pqulice` on the first commit.

### Steps

1. **[MUST]** Create the layout:

```
my-lib/
  .gitattributes  .gitignore  .mvn/jvm.config  .mvn/wrapper/  mvnw  mvnw.cmd
  .github/workflows/build.yml
  pom.xml  README.md  LICENSE.txt  REUSE.toml
  src/main/java/com/example/{MyLib,package-info}.java
  src/test/java/com/example/MyLibTest.java
```

2. **[MUST]** Configure the thin child POM (the parent does the heavy lifting):
```xml
<project>
  <modelVersion>4.0.0</modelVersion>
  <parent>
    <groupId>com.jcabi</groupId>
    <artifactId>parent</artifactId>
    <version><!-- Renovate --></version>
  </parent>
  <groupId>com.example</groupId>
  <artifactId>my-lib</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <properties>
    <maven.compiler.release>17</maven.compiler.release>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
  </properties>
</project>
```

3. **[MUST]** Add `package-info.java` (one-line Javadoc in every package):
```java
/**
 * Example objects.
 * @since 0.0.1
 */
package com.example;
```

4. **[MUST]** First object — interface first, `final` class, `private final` fields, code-free
   constructor, no getters:
```java
/** A greeting in a given language. */
public interface Greeting {
    /** @return The greeting text */
    String text();
}

/**
 * Greeting that never changes.
 * @since 0.0.1
 */
public final class FixedGreeting implements Greeting {
    private final String word;
    public FixedGreeting(final String wrd) { this.word = wrd; }
    @Override
    public String text() { return this.word; }
}
```

5. **[MUST]** First test — one statement, Hamcrest, package-private `final` class:
```java
final class FixedGreetingTest {
    @Test
    void greetsWithItsWord() {
        MatcherAssert.assertThat(
            "greeting text",
            new FixedGreeting("hi").text(),
            Matchers.equalTo("hi")
        );
    }
}
```

6. **[MUST]** Add an SPDX header (`SPDX-FileCopyrightText`, `SPDX-License-Identifier`) to every
   source/config file; set `.gitattributes` to `* text=auto eol=lf`.

7. **[MUST]** Write the README as the external contract: problem statement first paragraph, badges
   (EO, CI, coverage, license), install snippet, usage, Architecture, and the exact
   `mvn --errors --batch-mode clean install -Pqulice` command under "how to contribute".

8. **[MUST]** Add the CI workflow — one concern, least privilege, cache, timeout:
```yaml
name: build
on: [push, pull_request]
permissions:
  contents: read
concurrency:
  group: build-${{ github.ref }}
  cancel-in-progress: true
jobs:
  build:
    runs-on: ubuntu-latest
    timeout-minutes: 30
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-java@v4
        with: { distribution: temurin, java-version: '17', cache: maven }
      - run: mvn --errors --batch-mode clean install -Pqulice
```

### DoD
- [ ] `mvn --errors --batch-mode clean install -Pqulice` green locally and in CI.
- [ ] Every package has `package-info.java`; every file has an SPDX header.
- [ ] README documents install, usage, and the build command.
- [ ] `.gitattributes` pins `eol=lf`; `master` protected, PR-only.
- [ ] No `public static` behavior; the first object and test obey the one-assertion rule.

---

## Recipe 2 — Add a feature (ticket → failing test → objects → qulice → PR)

**Goal:** a small, reviewed, linted change tied to a ticket, arriving as a failing test first.

### Steps
1. **[MUST]** Open the ticket as a complaint with a minimal reproduction; branch named after it
   (`123-feature-name`). No BTW scope creep.
2. **[MUST]** Write a **failing test** expressing the desired public behavior and run it red:
   `mvn --errors --batch-mode test -Dtest=TheNewTest`.
3. **[MUST]** Design the objects before writing bodies: find the real entity the object represents
   and refuse `-er` names; keep interfaces ≤3 methods; every public method implements an interface
   method; inject collaborators through constructors (`new` only in secondary constructors); prefer
   a decorator over an `if` branch or a new method on an existing class.
4. **[MUST]** Implement the minimal code to green the test. No getters, no `null`, no statics,
   immutable fields, one `return` per method.
5. **[SHOULD]** Add a fake for any new interface (`Fake*`/`Mk*`, shipped with the interface when it
   is a legitimate object) instead of mocking in the test.
6. **[MUST]** Run the full gate locally: `mvn --errors --batch-mode clean install -Pqulice`.
7. **[MUST]** Keep it under one small PR (<50 hits-of-code for newcomers). Commit body references
   `#<ticket>`; push and open the PR; let the bot merge.

### Key snippet — decorator instead of an inline branch
```java
/** A document whose content is checked before it is returned. */
public final class CheckedDocument implements Document {
    private final Document origin;
    public CheckedDocument(final Document doc) { this.origin = doc; }
    @Override
    public byte[] content() {
        if (this.origin.content().length == 0) {
            throw new IllegalStateException("Empty document");
        }
        return this.origin.content();
    }
}
```

### DoD
- [ ] Ticket exists; branch and commits reference it.
- [ ] A failing test preceded the implementation (visible in history).
- [ ] Tests pass, one assertion each; Qulice green; coverage not reduced.
- [ ] New interface has a fake; public methods have Javadoc and `@Override`.
- [ ] PR is small, single-purpose, and bot-merged.

---

## Recipe 3 — Fix a bug (reproducing test → fix → regression PR)

**Goal:** turn a reported defect into a permanent, reproducing test and a minimal fix.

### Steps
1. **[MUST]** Reproduce the failure as a unit test in the same package, named after the behavior,
   not the bug number. Run red.
2. **[MUST]** If it cannot be reproduced, add a passing test proving the *intended* behavior and
   close the ticket with that evidence.
3. **[MUST]** Fix the cause, not the symptom. Keep the fix and the test in the same PR; do **not**
   refactor while fixing.
4. **[SHOULD]** If the fix grows beyond a micro-change, commit the `@Disabled` test + a
   `@todo #N:30min …` puzzle and file the refactor separately.
5. **[MUST]** Run the gate and confirm the regression test stays green:
   `mvn --errors --batch-mode clean install -Pqulice`.

### Key snippet — `@Disabled` reproduction + puzzle
```java
@Test
@Disabled
@todo #1234:30min Fix the off-by-one in RangeOf before enabling this test.
void includesTheUpperBound() {
    MatcherAssert.assertThat("range end", new RangeOf(1, 3).count(), Matchers.equalTo(3));
}
```

### DoD
- [ ] Reproducing test exists and was red before the fix.
- [ ] Fix and test in one PR; no unrelated refactor.
- [ ] Qulice green; full suite green; coverage not reduced.
- [ ] Commit message references the ticket; the reporter can close it.

---

## Recipe 4 — Refactor procedural code into EO objects

**Goal:** migrate statics/getters/mutable data into small immutable objects and decorators without
changing behavior, proven by tests before and after.

### Steps
1. **[SHOULD]** Freeze behavior: add characterization tests around the existing public behavior
   (one assertion each).
2. **[MUST]** Inventory the smells: `public static` methods, utility classes, getters/setters,
   mutable fields, `instanceof`/casting, `null` returns, `-er`/`*Util` names, public constants.
3. **[MUST]** Convert one concern per PR, in this order:
   - **statics → objects:** wrap each static behavior in a `final` class implementing an interface;
     the caller receives it via constructor.
   - **getter/setter pair → behavior:** replace `getX()`/`setX()` with a telling method returning a
     new object; mutations become `with(...)`.
   - **primitive/`if` → composition:** replace branching with a decorator or a `Filtered`/`Mapped`
     object.
   - **vendor static → wrapper:** hide a third-party utility behind a `Tk*`/`Dy*` object.
4. **[MUST]** Ratchet: ban the removed static API in the migrated package with `forbiddenapis`, and
   record the next target in a `@todo` puzzle.
5. **[MUST]** Run the characterization tests and the gate after each step.

### Key snippets
Statics to object:
```java
// before
public static String normalize(final String s) { return s.trim().toLowerCase(Locale.ENGLISH); }
// after
public final class Normalized implements Text {
    private final Text origin;
    public Normalized(final Text src) { this.origin = src; }
    @Override
    public String asString() { return this.origin.asString().trim().toLowerCase(Locale.ENGLISH); }
}
```
Imperative loop to declarative object:
```java
// before
final Collection<Integer> evens = new LinkedList<>();
for (final int n : numbers) { if (n % 2 == 0) { evens.add(n); } }
// after
final Collection<Integer> evens = new Filtered<>(numbers, n -> n % 2 == 0);
```
Ratchet: add `de.thetaphi:forbiddenapis` to the POM with a `forbidden-signatures.txt` listing the
banned static API, one migrated package at a time.

### DoD
- [ ] Characterization tests pass before and after; public behavior unchanged.
- [ ] No remaining `public static` behavior, getters/setters, or mutable fields in the touched package.
- [ ] Removed statics are banned by `forbiddenapis`; a `@todo` names the next package.
- [ ] Qulice green; each step is its own small PR.

---

## Recipe 5 — Introduce a quality gate into an existing repo

**Goal:** add Qulice + Jacoco + CI to a legacy repo and reach green by **ratcheting**, never by
disabling rules.

### Steps
1. **[MUST]** Inherit `com.jcabi:parent`; move cross-cutting plugin config there, keep domain deps
   and profiles in the child.
2. **[MUST]** Add Qulice to a profile bound to `verify`; add Jacoco thresholds to the POM.
3. **[MUST]** Collect the full violation list: `mvn --errors --batch-mode clean install -Pqulice`.
4. **[MUST]** Fix violations by **category**, smallest blast radius first — imports/whitespace →
   `final` modifiers → Javadoc → one-assert tests. Never lower the rule set.
5. **[SHOULD]** Where a rule must be boxed per file, use a narrow `@checkstyle`/`@SuppressWarnings`
   with a ticket-linked reason; remove any suppression that later covers nothing.
6. **[SHOULD]** Ratchet remaining legacy with `forbiddenapis` per migrated package rather than a
   global ban on day one.
7. **[MUST]** Make CI run the profile and act as the merge gate.

### Key snippets
Coverage gate:
```xml
<plugin>
  <groupId>org.jacoco</groupId>
  <artifactId>jacoco-maven-plugin</artifactId>
  <executions>
    <execution>
      <id>check</id>
      <goals><goal>check</goal></goals>
      <configuration>
        <rules><rule><element>BUNDLE</element><limits>
          <limit><counter>LINE</counter><value>COVEREDRATIO</value><minimum>0.65</minimum></limit>
        </limits></rule></rules>
      </configuration>
    </execution>
  </executions>
</plugin>
```
Narrow, reasoned exclusion (only when unavoidable):
`@SuppressWarnings("PMD.AvoidDuplicateLiterals") // @todo #2345:30min Deduplicate`

### DoD
- [ ] `mvn --errors --batch-mode clean install -Pqulice` green in CI.
- [ ] Rule set untouched; no blanket suppressions; each suppression has a ticket.
- [ ] Jacoco threshold enforced and met; CI is the merge gate on a read-only `master`.
- [ ] A `@todo` records the next legacy package to ratchet.

---

## Recipe 6 — Set up release automation (tag-driven + CI/Rultor + health check)

**Goal:** one tag produces a signed, reproducible release; release is not done until the endpoint
answers.

### Steps
1. **[MUST]** Keep source version `-SNAPSHOT`; never edit release versions by hand.
2. **[MUST]** Add `.rultor.yml` with a pinned image, release script, and secrets handling.
3. **[MUST]** Release script: validate the tag, set versions, commit, signed deploy.
```yaml
# .rultor.yml
docker: { image: yegor256/rultor-image:1.23.0, as_root: true }
merge:
  script: mvn --errors --batch-mode clean install -Pqulice
release:
  script: |
    set -e
    mvn versions:set -DnewVersion="${tag}"
    mvn --errors --batch-mode clean deploy -Pqulice,sonatype
```
4. **[MUST]** Validate the tag format before releasing:
```
[[ "${tag}" =~ ^[0-9]+\.[0-9]+\.[0-9]+$ ]] || { echo "bad tag"; exit 1; }
```
5. **[MUST]** Configure the `sonatype` profile with `maven-gpg-plugin` (sign), source+javadoc, and
   `nexus-staging-maven-plugin`; deploy from CI only.
6. **[SHOULD]** Add an `up.yml` workflow opening a PR that syncs docs/version from the latest tag.
7. **[SHOULD]** For apps, push to the PaaS remote then curl the live URL with retries:
```bash
for i in $(seq 1 30); do curl -fsS "https://app.example.com/health" && exit 0; sleep 10; done
exit 1
```
8. **[MUST]** Handle secrets by copy → commit → push → reset, with `trap EXIT` so nothing leaks into
   history.

### DoD
- [ ] Tag drives `versions:set`; no manual POM version edit.
- [ ] Every release is signed and downloadable; `master` remains read-only.
- [ ] CI runs lint + build + tests; Rultor is the only merger/deployer.
- [ ] Health check confirms the deployed endpoint; docs/version sync is automated.

---

## Recipe 7 — Review a PR in EO style

**Goal:** a constructive, rule-cited review that either accepts, holds firm, or appeals to the
architect — never a compromise.

### Steps
1. **[MUST]** Read the ticket first; confirm the PR is one small, single-purpose change.
2. **[MUST]** Walk the checklist and annotate each item pass/fail with `file:line`:
```
[ ] Ticket: every commit references #<ticket>; branch matches.
[ ] Tests: one assertThat per test; no @Before/fixtures; a test preceded the change.
[ ] Bug fix: ships a reproducing test.
[ ] Null: no null in/out; Null Object / collection / throw instead.
[ ] Statics: no public static behavior; no utility class/singleton.
[ ] Constructors: code-free; one primary ctor last; new only in secondary ctors.
[ ] Immutability: private final fields; mutations return a new object.
[ ] Names: no -er/-or titles; no get/set prefixes; nouns for builders, verbs for manipulators.
[ ] Contracts: every public method @Override's an interface; interface <=3 methods.
[ ] Inheritance: final classes; decorators, no implementation inheritance.
[ ] Exceptions: checked only; chain the cause; recover once at the top.
[ ] Style: Qulice green; Javadoc on types/methods/fields; <=100 cols; LF; no tabs.
[ ] Docs: README/version synced if behavior changed.
[ ] Scope: no refactor mixed into the fix; no BTW changes.
```
3. **[MUST]** Prove a problem before blocking: cite `file:line`, the rule (`[MUST]` or the Qulice rule
   name), and the smallest fix. The author need not prove the code good.
4. **[SHOULD]** Comment on the offending line (not PR top-level) and address the author by
   `@nickname`; suggest a concrete snippet where useful.
5. **[MUST]** Resolve disagreements exactly three ways: **accept**, **hold firm**, or **appeal to the
   architect** — never negotiate a middle ground.
6. **[SHOULD]** Let the bot merge and the reporter close the ticket.

### Key snippet — combined gate for a review
```
mvn --errors --batch-mode clean install -Pqulice
```
For a change with an HTTP surface, also verify the live endpoint returns 2xx before approving.

### DoD
- [ ] Each blocking comment cites a rule and a `file:line`; no taste-based blocks.
- [ ] Every checklist item is explicitly resolved.
- [ ] Merge path is PR-only via the bot; history untouched.
- [ ] After merge, docs/version sync (if needed) is a follow-up PR, never a silent edit.
