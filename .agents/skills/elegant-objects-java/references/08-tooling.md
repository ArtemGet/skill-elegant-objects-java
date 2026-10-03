# 08 — Tooling: pick-by-need catalogue

Audience: an AI coding agent setting up or reviewing an EO Java repository.
Rule of thumb: **a tool earns its place only if a build step or a test fails without it.**
Tags: **[MUST]** non-negotiable · **[SHOULD]** strong default · **[NICE]** case-by-case.
For every entry: *what / why / when to use / when NOT to use*.

---

## 0. The non-negotiable core stack

[MUST] Every EO Java repo ships exactly this baseline. Do not add a second tool where one of these
already covers the need.

| Layer | Choice | One-line reason |
|---|---|---|
| Language | **Java 17+** (`maven.compiler.release`) | Records, `var`, pattern matching; modern bytecode. |
| Build | **Maven** on **`com.jcabi:parent`** | Parent centralizes plugins, quality, release policy. |
| Test | **JUnit 5 (Jupiter) + Hamcrest** | One `assertThat(reason, actual, matcher)` per test. |
| Lint | **Qulice** | The enforceable EO quality wall; fails the build. |
| Coverage | **Jacoco** | Multi-counter thresholds in the POM. |
| Mutation | **PIT** | Proves tests detect change (libraries/critical cores). |
| API compat | **Revapi** | Semver gate on public API breaks. |
| Release bot | **Rultor** | The only merger/deployer; read-only `master`. |
| Deps | **Renovate** | Guard-railed dependency updates. |
| Licensing | **REUSE + SPDX** | Machine-checkable license on every file. |
| Puzzles | **PDD + 0pdd** | `@todo` puzzles become GitHub issues. |

[MUST] One command is the contract: `mvn --errors --batch-mode clean install -Pqulice`.

### Core dependency snippets

`pom.xml` skeleton (child POM stays thin):

```xml
<parent>
  <groupId>com.jcabi</groupId>
  <artifactId>parent</artifactId>
  <version><!-- latest, via Renovate --></version>
</parent>

<properties>
  <maven.compiler.release>17</maven.compiler.release>
  <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
</properties>

<dependencies>
  <dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter</artifactId>
    <scope>test</scope>
  </dependency>
  <dependency>
    <groupId>org.hamcrest</groupId>
    <artifactId>hamcrest</artifactId>
    <scope>test</scope>
  </dependency>
</dependencies>
```

[MUST] Compose `argLine` from an empty property so coverage agents can inject:

```xml
<properties>
  <argLine/>
</properties>
<plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-surefire-plugin</artifactId>
  <configuration>
    <argLine>@{argLine} -Xmx1g</argLine>
  </configuration>
</plugin>
```

[MUST] Keep the source version `-SNAPSHOT`; the release bot injects the tag with
`mvn versions:set -DnewVersion=${tag}`.

---

## 1. Build & toolchain

- **Java 17+ / `maven.compiler.release`** — *what:* pinned bytecode target. *why:* reproducibility.
  *use when:* every repo. *NOT when:* leaving it implicit or drifting from CI (the s3auth 1.8/21/11
  counter-example).
- **Maven** — *what:* declarative build. *why:* one non-interactive command for humans and CI.
  *use when:* always. *NOT when:* re-declaring what the parent already enforces.
- **`com.jcabi:parent`** — *what:* shared parent POM. *why:* centralizes plugins/quality/release.
  *use when:* always. *NOT when:* it is not reachable (then vendor the relevant config, never fork).
- **Maven wrapper / `.mvn/jvm.config`** — *what:* pinned Maven + JVM flags. *why:* reproducible
  builds. *use when:* newer repos. *NOT when:* older repos where CI installs Maven already.
- **`dependencyManagement` / BOM** — *what:* one version source. *why:* children omit versions.
  *use when:* multi-module or many deps. *NOT when:* pinning the same literal in every module.
- **`build-helper:reserve-network-port`** — *what:* random free port for ITs. *why:* no hardcoded
  ports. *use when:* any server IT. *NOT when:* a fixed port is genuinely required.
- **`maven-invoker-plugin`** — *what:* run standalone example/IT projects. *why:* proves consumers
  can use the artifact. *use when:* plugin/CLI-style projects. *NOT when:* a simple library with no
  usage samples.
- **`download-maven-plugin` / `exec-maven-plugin`** — *what:* fetch + run CLI-only tools.
  *why:* wraps binary distributions (DynamoDBLocal, Nutch). *use when:* no pure-Java alternative.
  *NOT when:* a pure-Java lib exists.
- **`license-maven-plugin`** — *what:* enforce the license header at `verify`. *why:* one license
  everywhere. *use when:* every repo. *NOT when:* never skip it.
- **`versions-maven-plugin`** — *what:* `versions:set` from the tag. *why:* automated release.
  *use when:* every tagged release. *NOT when:* editing POM versions by hand.
- **`nexus-staging-maven-plugin` + `maven-gpg-plugin` + source/javadoc** — *what:* signed Central
  release. *why:* public artifacts are verifiable. *use when:* public libraries. *NOT when:* internal
  apps (deploy via the PaaS instead).
- **`flatten-maven-plugin`** — *what:* flatten POM before deploy. *use when:* multi-module Central
  releases. *NOT when:* a minimal single-module lib.
- **`maven-assembly-plugin`** — *what:* fat/runnable JAR. *use when:* executables. *NOT when:*
  libraries (consumers pick deps).

[MUST] A library has zero non-`provided` runtime dependencies where possible; exclude transitive
duplicates (including self).

---

## 2. Static analysis, coverage, compatibility

- **Qulice** (Checkstyle + PMD + ErrorProne + custom EO rules) — *what:* the quality wall.
  *why:* enforces the [MUST] rules at build time. *use when:* every Java repo, mandatory in CI.
  *NOT when:* ever as an advisory report — it must fail the build.
  ```xml
  <!-- run: mvn --errors --batch-mode clean install -Pqulice -->
  ```
- **`forbiddenapis`** — *what:* ban an API in migrated packages. *why:* ratchet strict rules without
  breaking day-one legacy. *use when:* legacy migration, one package at a time. *NOT when:* banning
  globally at once.
- **Jacoco** — *what:* coverage + POM gates. *why:* unknown coverage is a maintainability sin.
  *use when:* always, multi-counter thresholds. *NOT when:* setting 100% — choose binding-but-reachable
  (~0.65 line is realistic).
- **PIT (pitest)** — *what:* mutation testing. *why:* proves assertions detect change. *use when:*
  libraries and critical cores (~75 mutation). *NOT when:* trivial glue or very slow suites.
- **Revapi** — *what:* binary/source API compatibility gate. *why:* semver must be honest. *use when:*
  public libraries. *NOT when:* apps with no external API. [SHOULD] Every accepted break carries a
  written `<justification>`.
- **ArchUnit** — *what:* architecture invariants (packages, single-parent hierarchy). *why:* encode
  design rules as tests. *use when:* large/multi-module codebases. *NOT when:* tiny single-module libs.
- **jtcop** — *what:* lints *test* code. *why:* test style is also a quality gate. *use when:* mature
  suites. *NOT when:* test style deliberately differs.
- **SonarCloud / Codacy / Infer / scancode** — *what:* dashboards + extra bug/security/license
  scanning. *use when:* larger projects wanting badges. *NOT when:* as a replacement for a failing
  build.
- **Simian** — *what:* duplication detection. *why:* copy-paste is a refactoring signal. *use when:*
  audits. *NOT when:* used to block a legitimately similar pair.
- **jPeek** — *what:* cohesion/coupling metrics. *use when:* research/refactoring audits. *NOT when:*
  as a runtime gate.
- **`hoc`** — *what:* Hits-of-Code metric. *why:* effort seen through change density. *use when:*
  dashboards/badges. *NOT when:* used to judge a contributor.

[MUST] When two analyzers contradict, disable exactly one rule with a written rationale; never
blanket-`@SuppressWarnings`. [MUST] Never leave a suppression that covers nothing.

---

## 3. Testing

- **JUnit 5 (Jupiter)** — *what:* test platform (tags, extensions, params). *why:* standard.
  *use when:* always. *NOT when:* never.
- **Hamcrest `MatcherAssert.assertThat`** — *what:* composable, declarative assertions. *why:* OO
  matchers beat `Assert.*`. *use when:* always, with a reason string. *NOT when:* raw JUnit asserts
  (Qulice forbids).
- **cactoos-matchers / jcabi-matchers / `XhtmlMatchers`** — *what:* OO matchers (`HasValue`,
  `IsText`, XPath). *use when:* OO assertions, XML/HTTP. *NOT when:* plain value comparison.
- **Fakes (`Fake*` / `Mk*` / `*Mocker` / `Fk*`)** — *what:* real reusable implementations of an
  interface. *why:* tests assert behavior, not interactions. *use when:* always prefer over mocks;
  ship in `src/main` when it is a legitimate object. *NOT when:* hiding genuinely internal classes.
- **Mockito** — *what:* interaction mocking. *use when:* a rare spy case only. *NOT when:* as the
  default — prefer fakes.
- **jqwik** — *what:* property-based testing. *use when:* parsers/serializers/escaping. *NOT when:*
  example-only unit tests.
- **JMH** — *what:* micro-benchmarks. *use when:* performance-critical code. *NOT when:* as a regular
  CI gate (noisy).
- **DynamoDBLocal / H2 + reserved ports** — *what:* a real dependency for persistence ITs. *use when:*
  persistence `*ITCase`. *NOT when:* unit tests.
- **`MkContainer` / `FtRemote` (jcabi-http)** — *what:* a real HTTP stack in tests. *use when:* HTTP
  clients/servers. *NOT when:* pure logic tests.
- **`@TempDir` / junit-pioneer** — *what:* inject temp paths/env. *use when:* integration-ish unit
  tests. *NOT when:* trivial tests.

[MUST] One statement per test; no `@Before`; no shared fixtures; no logging in tests.
[MUST] A bug fix ships with a reproducing test. [SHOULD] Guard optional tooling with `Assumptions`.

---

## 4. Runtime libraries (the OO stack)

- **Cactoos** — *what:* OO primitives replacing Guava/Commons (`ListOf`, `BytesOf`, `Sticky`,
  `SyncScalar`). *why:* declarative composition, zero runtime deps. *use when:* default
  primitive/IO library. *NOT when:* wrapping it back into getters or adding Guava alongside.
- **Takes** — *what:* OO web framework (`Take`/`Request`/`Response`). *use when:* web apps, REST,
  chat-ops. *NOT when:* a heavy Spring-MVC shop (migration risk).
- **jcabi-http / jcabi-xml / jcabi-jdbc / jcabi-dynamo / jcabi-s3 / jcabi-ssh / jcabi-manifests /
  jcabi-log** — *what:* OO wrappers over HTTP/XML/SQL/Dynamo/S3/SSH/manifests/logging. *why:* hide
  vendor types behind objects. *use when:* persistence/integration seams. *NOT when:* used once —
  wrap the JDK call yourself.
- **Xembly** — *what:* imperative XML generation (`Iterable<Directive>`). *use when:* build XML
  without JAXB/getters. *NOT when:* parsing XML (use jcabi-xml).
- **Saxon-HE** — *what:* XSLT/XPath 2.0+ (the JDK is 1.0). *use when:* transformations need 2.0.
  *NOT when:* plain 1.0 stylesheets.
- **AWS SDK v2/v1** — *what:* S3/CloudWatch/DynamoDB. *use when:* apps using AWS — hide behind
  interfaces (`Dy*`). *NOT when:* domain code — never let AWS types leak.
- **jcabi-aspects** — *what:* AOP decorators (`@Immutable`, `@Loggable`, `@RetryOnFailure`).
  *use when:* cross-cutting concerns where a visible decorator is impractical. *NOT when:* a visible
  decorator would do (the book prefers it).

### Essential dependency snippet

```xml
<dependency>
  <groupId>org.cactoos</groupId>
  <artifactId>cactoos</artifactId>
  <version><!-- Renovate --></version>
</dependency>
<dependency>
  <groupId>com.jcabi</groupId>
  <artifactId>jcabi-http</artifactId>
  <version><!-- Renovate --></version>
</dependency>
```

---

## 5. Process / repo / CI tooling

- **Rultor** — *what:* chat-ops merge + release bot on a pinned Docker image. *why:* one controlled
  write path. *use when:* the only deployer; `master` is read-only. *NOT when:* letting a human push
  to `master`.
- **PDD + 0pdd** — *what:* `@todo #N:30min …` puzzles become issues. *use when:* every repo. *NOT
  when:* bare `TODO`/`FIXME`.
- **Renovate** — *what:* rule-based dependency updates. *use when:* every repo. *NOT when:*
  Dependabot.
- **REUSE + SPDX + `copyrights`** — *what:* machine-checkable licensing. *use when:* every file in
  every repo. *NOT when:* leaving one file without a header.
- **GitHub Actions** — *what:* the PR gate (lint + build + tests + coverage). *why:* separate from
  the release bot. *use when:* every repo. *NOT when:* conflated with Rultor.
- **Codecov / SonarCloud** — *what:* coverage/quality visibility. *use when:* larger projects, gate
  with `fail_ci_if_error`. *NOT when:* a substitute for POM thresholds.
- **`hoc` / `tdx` / 0pdd badges** — *use when:* README dashboards. *NOT when:* vanity-only.
- **AI agents for PDD chores** — *what:* bug rewording, small refactor PRs, docs sync. *use when:*
  one small concern per PR. *NOT when:* a large unscoped change.

---

## 6. Decision table (situation → tool)

| Situation | Pick | Do NOT pick |
|---|---|---|
| New repo build | Maven + `com.jcabi:parent` + wrapper | Gradle, ad-hoc scripts |
| Enforce EO rules | Qulice in `verify` | advisory lint report |
| Public method behavior | test with one `assertThat` | `System.out`, logging |
| Need a collaborator double | `Fake*`/`Mk*`/`Fk*` in `src/main` | Mockito as default |
| Legacy code from 3rd party | wrapper object (`Tk*`, `Dy*`) | leak vendor types |
| Parse/collect primitives | Cactoos (`ListOf`, `Filtered`, `Mapped`) | Guava/Commons in new code |
| HTTP request in prod | jcabi-http / Takes | hand-rolled `HttpURLConnection` |
| XML output | Xembly | JAXB marshalling |
| XSLT 2.0+ | Saxon-HE | JDK XSLT (1.0 only) |
| SQL in domain | SQL-speaking objects | ORM / ActiveRecord |
| Persistence IT | DynamoDBLocal/H2 + reserved port | mocks of the DB |
| HTTP client/server test | `MkContainer`/`FtRemote` | empty mocks |
| Serializer/parser tests | jqwik properties | 2 examples only |
| Hot path | JMH benchmark | stopwatch in CI |
| Coverage gate | Jacoco in POM | "we'll check later" |
| Mutation gate | PIT (~75) | trusting line coverage alone |
| Public API break | Revapi + `<justification>` | silent semver |
| Architecture rule | ArchUnit test | wiki page nobody reads |
| Test-code lint | jtcop | manual review only |
| Duplication | Simian audit | copy-paste refactor |
| Dependency freshness | Renovate (pinned + rules) | dynamic ranges everywhere |
| Release | tag → `versions:set` → signed deploy | manual POM edit |
| Merge/deploy | Rultor PR-only | force-push to `master` |
| License check | REUSE + SPDX + `license-maven-plugin` | one LICENSE file only |
| Backlog tracking | PDD/0pdd puzzles | informal TODO comments |

---

## 7. What to AVOID (and the replacement)

[MUST] Do not introduce these into EO code. Each is an anti-pattern with a concrete substitute.

| Avoid | Why | Replace with |
|---|---|---|
| **DI containers** (Guice, Spring, Dagger) | hides composition; field injection makes incomplete mutable objects | constructor composition + `new` at the edge |
| **ORM** (Hibernate, JPA) | SQL becomes a leaky detail; entities are data bags | SQL-speaking objects behind an interface |
| **Lombok-heavy code** (`@Data`, `@Builder`, `@Setter`) | generates getters/setters/mutability | hand-written `final` value objects; at most `@EqualsAndHashCode`/`@ToString` (`provided`) |
| **Builders** (mutable fluent) | a growing arg list means missing collaborators | secondary constructors / `with(...)` returning a new object |
| **Guava / Commons as public API** | leaks vendor types into contracts | Cactoos/jcabi, or a dedicated wrapper class |
| **PowerMock / static mocking** | mocks statics/constructors; proves procedural design | remove the static; inject via constructor |
| **JAXB for output** | behavior-carrying annotations; DTO marshalling | Xembly (build) / jcabi-xml (parse) |
| **`java.util.Optional`** | not a Null Object; hides absence in a wrapper | Null Object, empty collection, or throw |
| **Public constants / enums** | global hard coupling | micro-classes |
| **`-er` job titles** (`Manager`, `Helper`, `Controller`) | describes procedures, not entities | name what the object *is* |
| **Static utility classes / singletons** | global state, un-mockable | inject collaborators through constructors |
| **`instanceof` / reflection / casting** | discriminates by type; hidden coupling | polymorphism / decorators |

[NICE] If a dependency is truly required, pin it in `dependencyManagement`, keep it `provided` where
possible, and wrap its static API in a small object so domain code never touches vendor statics.

---

## 8. Setup checklist (agent)

[MUST] 1. Java 17+ `release` == CI JDK == README claim.
[MUST] 2. `com.jcabi:parent` in `<parent>`; child POM only domain deps + profiles.
[MUST] 3. `pom.xml` gates: Jacoco thresholds, PIT (libraries), Revapi (public API).
[MUST] 4. Profiles by purpose (`qulice`, `jacoco`, `deep`, `sonar`, `site`), never by environment.
[MUST] 5. `mvn --errors --batch-mode clean install -Pqulice` documented in the README.
[MUST] 6. REUSE + SPDX header on every source/config file.
[SHOULD] 7. One small workflow per concern, each with `permissions: contents: read`,
`concurrency`, `timeout-minutes`, and a cache keyed on `hashFiles('**/pom.xml')`.
[SHOULD] 8. Renovate config with rules for pins that must not float.
[MUST] 9. Rultor as the only merger/deployer; `master` read-only.
[NICE] 10. Wrap every vendor static in an object; keep vendor types behind `Dy*`/`Tk*` classes.
