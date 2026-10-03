# 02 — Testing in Elegant Objects Java

Operating manual for an AI agent writing/repairing tests in an EO Java project. Tags: **[MUST]**
(build fails), **[SHOULD]** (default; deviate with a written reason), **[NICE]** (review guidance).
Sources: `knowledge/distilled-rules.md` §7; `knowledge/elegant-objects-java.md` (`B7.*`, `Atq.*`).

> A test is part of the class (`B7.1`); a missing test is a bug (`Atq.19`); a flaky test is a bug (`Atq.20`). Tests are the warranty on previously paid-for code, not optional homework.


## 1. Test-driven & bug-driven development
EO default is **bug-driven**, not test-first; a clear new requirement may be test-first, a defect must be.
**RULES**
- **[MUST]** Begin every bug fix with a failing test that reproduces it (`B7.12`, `Atq.24`): red → fix
  → green.
- **[MUST]** No behaviour-changing PR may add zero tests (`Atq.35`); replace debug sessions with a
  reproducing test (`Atq.1`).
- **[SHOULD]** Test to find errors, not confirm correctness (`Atq.18`); ship a test even for a typo —
  the real defect is the missing test (`Atq.75`).

| Smell | Fix |
|---|---|
| "Tested manually" then committed | Add the test, watch it fail, then fix |
| Bug fixed with no test added | Block the PR (`Atq.35`) |

```java
// BEFORE: fix applied, no proof it was ever broken
int fibo(int n) { return n <= 1 ? n : fibo(n - 1) + fibo(n - 2); }

// AFTER: failing test first, then the fix
@Test void calculates23rdFibonacci() {
    assertThat("23rd Fibonacci", new Fibo(23).value(), equalTo(28657));
}
```

## 2. One-statement tests (arrange by composition + one `assertThat`)
A test body is exactly one statement: a single `MatcherAssert.assertThat(reason, actual, matcher)`.
**RULES**
- **[MUST]** Exactly one `assertThat` per method (Qulice `UnitTestContainsTooManyAsserts`); use
  Hamcrest, never `Assert.*` (`B7.3`, `Atq.42`).
- **[MUST]** Pass a human-readable reason first; no flow control in the body (`Atq.29`, `Atq.21`).
- **[SHOULD]** Arrange inline via composition; assert only what you care about (`Atq.23`, `Atq.42`).

| Smell | Fix |
|---|---|
| Two or more `assertThat` calls | Split into two tests, one assertion each |
| `String r = call(); assertThat(r, ...)` | Inline: `assertThat("reason", call(), matcher)` |

```java
// BEFORE: setup statements, no reason
Phrases p = new Phrases("Hello, world!");
assertNotNull(p.greetings().iterator().next());

// AFTER: one composed statement, one assertion, reason
@Test void countsSimpleGreetings() {
    assertThat("Total count of greetings",
        new Phrases("Hello, world!").greetings().count(), equalTo(1));
}
```

## 3. No fixtures, no `@Before`
No lifecycle hooks, no shared fields or constants; each test builds its own literals.
**RULES**
- **[MUST]** Never use `@BeforeEach`/`@BeforeAll` to prepare data (`Atq.21`).
- **[MUST]** No fields in a test class; give each test its own literals — duplication is desired
  (`Atq.22`).
- **[SHOULD]** Inject prerequisites via a JUnit 5 `ParameterResolver`, and ultimately fake objects
  (`Atq.36`).

| Smell | Fix |
|---|---|
| `@BeforeEach void setUp()` | Construct the subject inside each test |
| `private static final String MSG = "x"` | Use a different local literal per test |

```java
// BEFORE: shared field couples tests
private Phrases phrases;
@BeforeEach void setUp() { this.phrases = new Phrases("Hi!"); }

// AFTER: each test stands alone
@Test
void countsTwoSentences() {
    assertThat("count", new Phrases("Hi! Bye!").count(), equalTo(2));
}
```

## 4. Fakes over mocks
Never use Mockito-style mocks. Every interface needing a double ships a `Fake`/`Fk*`/`Mk*` in
`src/main` when it is a legitimate reusable object (`B7.4`–`B7.8`).
**RULES**
- **[MUST]** No mocking framework in business-logic tests; every interface ships its fake (`B7.4`,
  `B7.5`).
- **[MUST]** Tests assert public behaviour only, never internal interactions (`B7.7`).
- **[MUST]** When the interface changes, its fake changes; the tests do not (`B7.8`).
- **[SHOULD]** Put reusable fakes in `src/main` (`Atq.23`) and test your doubles (`Atq.26`).

| Smell | Fix |
|---|---|
| `Mockito.mock(Foo.class)` + `verify()` | Hand-write `FakeFoo implements Foo` |
| Assertion on "was method called" | Assert observable output of the public method |

```java
// BEFORE: mock + interaction verification
Foo foo = Mockito.mock(Foo.class);
when(foo.value()).thenReturn(42);
verify(foo).value();

// AFTER: fake object, output assertion only
final class FakeFoo implements Foo { @Override public int value() { return 42; } }
assertThat("total", new Bar(new FakeFoo()).total(), equalTo(42));
```

## 5. Test naming & class layout (`*Test` vs `*ITCase`)
`FooTest` runs under surefire (`test`); `FooITCase` under failsafe (`integration-test → verify`)
(`Atq.5`, `Atq.56`).
**RULES**
- **[MUST]** `FooTest` validates `Foo`; `FooITCase` exercises `Foo` against a real dependency
  (`Atq.5`).
- **[MUST]** Test classes are `final` and package-private (Qulice `JUnitTestClassShouldBeFinal`).
- **[MUST]** Method names are behaviour sentences; no `test` prefix, no `test1`/`check` (`Atq.6`).

| Smell | Fix |
|---|---|
| `FooIntegrationTest` | Rename `FooITCase`, move under `it/`, wire failsafe |
| `public class FooTest` | Make it `final`, package-private |

```xml
<plugin><artifactId>maven-failsafe-plugin</artifactId>
  <configuration><includes><include>**/*ITCase.java</include></includes></configuration>
  <executions><execution><goals><goal>integration-test</goal><goal>verify</goal></goals></execution></executions>
</plugin>
```

## 6. Coverage & mutation gates
Coverage is gameable by assertion-free tests; mutation proves the tests detect real changes. Both live
in the POM and fail the build (`Apa.26`, `Atq.50`; realistic bar ~0.65 line / ~75 mutation).
**RULES**
- **[MUST]** Jacoco line-coverage threshold in the POM; build fails below it (`Apa.26`).
- **[MUST]** PIT mutation threshold in the POM for libraries (`Atq.50`).
- **[MUST]** Compose `argLine` from an empty property (`<argLine/>` + `@{argLine} …`) so agents inject
  (`knowledge/elegant-objects-java.md` §H).

| Smell | Fix |
|---|---|
| Assertion-free tests inflate coverage | Add real matchers; enable PIT to expose them |
| Coverage gate dropped in a PR | Revert; fix the tests instead |

```xml
<properties><argLine/></properties>
<plugin><artifactId>jacoco-maven-plugin</artifactId><executions>
  <execution><goals><goal>prepare-agent</goal></goals></execution>
  <execution><id>check</id><goals><goal>check</goal></goals>
    <configuration><rules><rule><element>BUNDLE</element><limits><limit>
      <counter>LINE</counter><value>COVEREDRATIO</value><minimum>0.65</minimum>
    </limit></limits></rule></rules></configuration></execution>
</executions></plugin>
<plugin><artifactId>pitest-maven</artifactId>
  <configuration><mutationThreshold>75</mutationThreshold></configuration></plugin>
```

## 7. Fast vs deep tests (`@Tag`)
Tag unit tests `@Tag("fast")` (<20 ms, no I/O) and integration tests `@Tag("deep")` (real resources);
run fast locally, deep on CI (`Atq.30`, `Atq.54`).
**RULES**
- **[MUST]** Tag slow/real-resource tests `@Tag("deep")`; never delete them to speed the build
  (`Atq.80`).
- **[SHOULD]** Default surefire runs fast; CI additionally runs deep (`Atq.54`).
- **[SHOULD]** Use JUnit 5 native tags, not home-grown categorisation (`Atq.69`).

| Smell | Fix |
|---|---|
| Whole suite slow on every save | Move I/O tests behind `@Tag("deep")` |
| Slow tests deleted | Tag them and run on CI instead |

```java
// BEFORE: untagged, runs locally, slow
@Test void readsFromManyFiles() throws IOException { ... }

// AFTER
@Test @Tag("fast") void readsSomeData() throws IOException { ... }
@Test @Tag("deep") void readsFromManyFiles(@TempDir Path tmp) throws IOException { ... }
```

## 8. Integration / HTTP / persistence testing
Exercise real dependencies, not mocks: a real HTTP server on a random port, embedded
`DynamoDBLocal`/H2 on a reserved port, provisioned via Maven profiles (`Atq.38`, `Adt.41`).
**RULES**
- **[MUST]** Test HTTP against a real socket/server on a random port; never mock the HTTP stack
  (`Atq.38`).
- **[MUST]** Use `*ITCase` + failsafe for anything touching the DB/network (`Atq.56`).
- **[SHOULD]** Use `jcabi-http` `MkContainer`/`MkGrizzlyContainer` and embedded `DynamoDBLocal`/H2
  (`Atq.67`, `Adt.41`).

| Smell | Fix |
|---|---|
| HTTP response stubbed by a mock | `MkGrizzlyContainer` serving a queued answer |
| Tests share one fixed port | Use a random/reserved port per run |

```java
// BEFORE: mocked HTTP client, no real stack
when(client.fetch()).thenReturn(new Answer("hello"));

// AFTER: real mock server on a random port
final MkContainer container = new MkGrizzlyContainer()
    .next(new MkAnswer.Simple("hello, world!")).start();
try {
    assertThat("body", new JdkRequest(container.home()).fetch().body(),
        containsString("hello"));
} finally { container.stop(); }
```

## 9. Property-based tests (jqwik)
For parsers/serializers, generate many inputs with **jqwik**; a property must hold for *all* generated
inputs, and a counterexample is a bug (`Atq.67`).
**RULES**
- **[MUST]** Round-trip property `parse(print(x)) == x` for parsers/serializers (`Atq.67`).
- **[SHOULD]** Use `@Property` + `@ForAll`; keep each property to one assertion (`B7.3`).
- **[SHOULD]** Cross-check alternative implementations against the same property
  (`knowledge/distilled-rules.md` §7).

| Smell | Fix |
|---|---|
| Hand-written table of 50 examples | Replace with a `@Property` + generators |
| Parser tested only on happy input | Add a round-trip property over generated text |

```java
// BEFORE
@Test void parsesSimple() { assertThat(parse("a=1").get("a"), equalTo("1")); }

// AFTER
@Property
void roundTrips(@ForAll String key, @ForAll String value) {
    assertThat("round-trip", parse(print(key, value)), hasEntry(key, value));
}
```

## 10. Concurrency tests (latch)
A `ConcurrentHashMap` field does not make a compound operation atomic; prove thread-safety with a
latch-driven parallel test that forces overlap (`Atq.4`, `Atq.31`).
**RULES**
- **[MUST]** Synchronize compound check-then-act operations and cover them with a parallel test
  (`Atq.4`, `Atq.79`).
- **[MUST]** Use a `CountDownLatch` so all threads race at once (`Atq.31`).
- **[SHOULD]** Prefer Cactoos `Threads`/`RunsInThreads`; keep threading out of the core class as a
  `Sync*` decorator (`Atq.32`, `Acob.16`).

| Smell | Fix |
|---|---|
| Sequential `for` loop calling the method | Latch + `ExecutorService` to force overlap |
| Trusting `ConcurrentHashMap` | Guard the compound op and test it in parallel |

```java
// BEFORE: sequential, proves nothing
for (int t = 0; t < 10; ++t) { assertThat(books.add("x"), equalTo(t + 1)); }

// AFTER: all threads race, then assert the invariant
final CountDownLatch latch = new CountDownLatch(1);
for (int t = 0; t < 10; ++t) {
    futures.add(service.submit(() -> { latch.await(); return books.add("Book"); }));
}
latch.countDown();
assertThat("unique ids", futures.stream().map(Future::get).distinct().count(), equalTo(10L));
```

## 11. Flaky tests & `Assumptions`
A flaky test is a bug (`Atq.20`). Never delete or blindly `@Disabled` it; guard optional
tooling/resources with `Assumptions` (`CompilerTest.java:154-168`).
**RULES**
- **[MUST]** File a defect for any intermittently failing test; do not ignore a red build (`Atq.20`).
- **[MUST]** Use `Assumptions.assumeTrue(...)` for optional tools/resources; never delete the test.
- **[SHOULD]** Fix the root cause (time, port, ordering, shared state), and isolate non-determinism
  with `@TempDir` and random ports.

| Smell | Fix |
|---|---|
| `@Disabled` on a flaky test forever | Root-cause it, or guard with `Assumptions` |
| `Thread.sleep` before assertion | Wait on a condition/watcher; `@TempDir` per test |

```java
// BEFORE: fails when the compiler is absent; or was deleted
@Test void compilesSnippets() { new Compiler().compile("code"); }

// AFTER: skip honestly when the tool is unavailable
@Test
void compilesSnippets() throws Exception {
    Assumptions.assumeTrue(new File("/usr/bin/javac").exists());
    assertThat("compilation result", new Compiler().compile("code"), not(empty()));
}
```

## 12. `@Disabled` test + `@todo` puzzle as a bug report
Ship a `@Disabled` test reproducing the defect plus a PDD `@todo #N` puzzle; the puzzle becomes a
ticket and the PR doubles as reproduction (`Atq.33`, `Atq.34`).
**RULES**
- **[MUST]** Report a bug as a disabled (or failing) test + `@todo #N` puzzle, not prose alone
  (`Atq.34`).
- **[MUST]** No bare `TODO`/`FIXME`; use `@todo #N:30min …` (`Apa.39`–`Apa.42`).
- **[SHOULD]** The puzzle names the concrete defect and expected value (`Atq.11`), so the test can be
  enabled later (`Atq.58`).

| Smell | Fix |
|---|---|
| Issue text "feature X is broken" | Disabled test reproducing X + `@todo #N` puzzle |
| `// TODO fix this` | `// @todo #42 fix ...` |

```java
// BEFORE: prose-only report, no reproduction
// "fibo(23) is wrong"

// AFTER: disabled test + puzzle
// @todo #42 fibo(23) returns 17711 but should return 28657
@Disabled("reproduces #42") @Test void calculates23rdFibonacci() {
    assertThat("23rd Fibonacci", new Fibo(23).value(), equalTo(28657));
}
```

## 13. Separate PR for tests
Put the (disabled) test in its own PR reviewed for *intent*, before the implementation PR — keeping
requirements review separate from implementation review (`Atq.33`).
**RULES**
- **[MUST]** Never modify or disable a failing test merely to make a build pass (`Atq.74`).
- **[SHOULD]** PR 1 adds the disabled test (reviewers validate the contract); PR 2 fixes code without
  touching the test (`Atq.33`).
- **[SHOULD]** Keep the coverage gate intact while tests are disabled (`Atq.58`).

| Smell | Fix |
|---|---|
| Fix + test edits in one commit | Split into a test PR and a fix PR |
| Test weakened to pass | Revert the test edit; fix the production code |

```text
// BEFORE (one mixed PR)
"fix fibo + disable slow test + bump deps"

// AFTER (two PRs)
PR #1: add @Disabled calculates23rdFibonacci + @todo #42   (intent reviewed)
PR #2: fix Fibo so the test can be enabled                 (no test edits)
```

## 14. JUnit 5 + Hamcrest setup
Keep JUnit 5 (Jupiter) + Hamcrest `test`-scope; no plain JUnit assertions. Add jqwik where properties
are worth it.
```xml
<dependencies>
  <dependency><groupId>org.junit.jupiter</groupId><artifactId>junit-jupiter</artifactId>
    <version>5.10.2</version><scope>test</scope></dependency>
  <dependency><groupId>org.hamcrest</groupId><artifactId>hamcrest</artifactId>
    <version>2.2</version><scope>test</scope></dependency>
  <dependency><groupId>net.jqwik</groupId><artifactId>jqwik</artifactId>
    <version>1.9.1</version><scope>test</scope></dependency>
</dependencies>
```
**Full example test class**
```java
package com.example;

import org.hamcrest.MatcherAssert;
import org.hamcrest.Matchers;
import org.junit.jupiter.api.Tag;
import org.junit.jupiter.api.Test;

final class PhrasesTest {

    @Test
    @Tag("fast")
    void countsSimpleGreetings() {
        MatcherAssert.assertThat(
            "Total count of greetings",
            new Phrases("Hello, world!").greetings().count(),
            Matchers.equalTo(1)
        );
    }
}
```

## 15. `@Tag` + profile config sketch
Default surefire runs `fast`; the `deep` profile also runs slow/deep tests, wired through `<groups>`
(`Atq.30`, `Atq.54`).
```xml
<plugin>
  <artifactId>maven-surefire-plugin</artifactId>
  <configuration><groups>fast</groups></configuration>
</plugin>
```
```xml
<profiles><profile>
  <id>deep</id>
  <properties><groups>fast,deep</groups></properties>
  <build><plugins><plugin>
    <artifactId>maven-surefire-plugin</artifactId>
    <configuration><groups>${groups}</groups></configuration>
  </plugin></plugins></build>
</profile></profiles>
```
`mvn test` = fast loop; `mvn clean install -Pqulice,deep` = CI (lint + deep tests).

## Test definition of done
A change is not done until **all** of these are true:
- [ ] A test would have failed before the change (red → green), or a `@Disabled` test + `@todo #N`
      puzzle reproduces the reported defect.
- [ ] Every test method is one statement: exactly one `MatcherAssert.assertThat(reason, actual,
      matcher)` — one semantic assertion.
- [ ] No `@Before`/`@BeforeEach`/`@BeforeAll`, no fields, no shared constants; each test builds its
      own literals.
- [ ] No mocking framework; any double is a `Fake`/`Fk*`/`Mk*` in `src/main`, moved with its
      interface.
- [ ] Unit tests named `FooTest` (surefire, no network/disk/DB); integration tests `FooITCase` under
      `it/` (failsafe).
- [ ] Slow/real-resource tests carry `@Tag("deep")`, fast tests `@Tag("fast")`; HTTP/persistence use a
      real server on a random port (`MkGrizzlyContainer`, DynamoDBLocal/H2), not mocks.
- [ ] Parsers/serializers have a property-based (jqwik) round-trip test; concurrency claims have a
      latch-driven parallel test.
- [ ] No flaky tests; optional tools guarded with `Assumptions`, not deleted.
- [ ] Coverage (Jacoco) and mutation (PIT) gates pass; `argLine` is injectable.
- [ ] No test was weakened or disabled just to make the build green; tests reviewed separately from
      the fix where practical.
- [ ] `mvn --errors --batch-mode clean install -Pqulice` is green, including Qulice's one-assert and
      final-test-class checks.
