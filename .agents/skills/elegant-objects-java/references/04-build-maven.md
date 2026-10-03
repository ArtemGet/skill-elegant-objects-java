# 04 — Build & Maven

Reference for an AI agent configuring or reviewing the Maven build of an Elegant Objects
(EO) Java project. Tags: **[MUST]** build fails without it · **[SHOULD]** strong default ·
**[NICE]** improves reproducibility/elegance. Distilled from `knowledge/distilled-rules.md`
§10 and the POMs in `knowledge/elegant-objects-java.md` §B/§C and `knowledge/elegant-objects-java.md` §C.

---

## 1. The one build command

**[MUST]** Exactly one canonical, non-interactive command; CI, the README and every bot use
the *same* string:

```bash
mvn --errors --batch-mode clean install -Pqulice
```

`--errors` shows the real error; `--batch-mode` removes prompts/ANSI; `clean install` builds,
tests, packages, installs; `-Pqulice` makes static analysis mandatory. **[SHOULD]** The local
loop may drop `-Pqulice` (`mvn test`); the CI gate may not. Never document a second command.

## 2. Parent POM strategy

**[MUST]** Inherit a shared policy parent, `com.jcabi:parent` (`0.73.x`) — see the `<parent>`
block in §12 — and keep the repo POM thin: only domain dependencies, dependencyManagement and
profiles. The parent centralizes plugin versions, compiler config, surefire/failsafe wiring, license
checks and Qulice; a parent is the fix for copy-pasted plugin config (`Adt.74`). **[MUST]**
Never redefine a parent-managed plugin version unless you must — pin the exception and comment
*why*. **[MUST]** Multi-module aggregators are `packaging=pom`, `<modules>` in dependency
order. **[SHOULD]** Keep the repo POM ~100–600 lines.

## 3. Pin the toolchain: release == CI JDK == docs

**[MUST]** `maven.compiler.release`, the CI JDK and the README JDK must be the same number
(the s3auth drift POM 1.8 / CI 21 / README 11 is the counter-example). The floor is enforced
in the build (see the enforcer block in §12), not only in prose. **[MUST]** Prefer
`maven.compiler.release` (one knob, refuses post-release APIs) over `source`+`target`, and pin
`project.build.sourceEncoding`/`project.reporting.outputEncoding` to `UTF-8` (see §12).

## 4. Versioning & release (`-SNAPSHOT` + `versions:set`)

**[MUST]** Keep the in-tree version `X.Y-SNAPSHOT` forever. A release is a tag-driven
`versions:set`; never hand-edit the version to a release number in a normal PR:

```bash
# release bot / release.sh, run only for a semver tag
[[ "${tag}" =~ ^[0-9]+\.[0-9]+\.[0-9]+$ ]] || exit 1
mvn -ntp versions:set -DnewVersion="${tag}" --quiet   # inject the tag version
git commit -am "${tag}"
mvn clean package -Pqulice --errors --batch-mode
```

**[SHOULD]** Bump the README-pinned version in the same scripted commit, or open a follow-up
`up.yml` PR from the new tag (doc/version sync — §05). **[NICE]** `flatten-maven-plugin`
before deploy so consumers don't see parent indirection.

## 5. dependencyManagement / BOM

**[MUST]** Centralize every dependency version: library/aggregator parents declare a
`<dependencyManagement>`; children omit `<version>` (comment `<!-- version from parent POM -->`).

```xml
<dependencyManagement>
  <dependencies>
    <!-- upstream BOM: versions come from that project -->
    <dependency>
      <groupId>org.junit</groupId>
      <artifactId>junit-bom</artifactId>
      <version>6.1.3</version>
      <type>pom</type><scope>import</scope>
    </dependency>
    <!-- local managed version: children omit <version> -->
    <dependency>
      <groupId>org.takes</groupId>
      <artifactId>takes</artifactId>
      <version>1.26.0</version>
    </dependency>
  </dependencies>
</dependencyManagement>
```

**[MUST]** No version ranges except explicitly trusted authors (`Ada.31`); pin exact.
**[SHOULD]** Let Renovate move pins; guard the ones that must not float (§05).

## 6. Dependency hygiene

**[MUST]** A library declares **zero non-`provided` runtime dependencies** where possible —
its API must not drag a graph into consumers. **[MUST]** Exclude transitive duplicates,
including self, so the graph stays flat:

```xml
<dependency>
  <groupId>org.takes</groupId>
  <artifactId>takes</artifactId>
  <exclusions>
    <exclusion>
      <groupId>commons-logging</groupId><artifactId>commons-logging</artifactId>
    </exclusion>
  </exclusions>
</dependency>
```

**[MUST]** Keep vendor types behind a `Dy*`/`Tk*` wrapper; never leak a third-party type
through the public API (`B11.5`). **[SHOULD]** Scope test-only deps `test`, compile-only
helpers `provided`; declare `opentest4j` explicitly even when transitive — Qulice's type-check
classpath only sees directly declared deps. **[NICE]** Publish fakes when they are part of the
contract (`Acoa.38`).

## 7. Profiles by purpose (never by environment)

**[MUST]** Profiles are build *dials* named by what they do, not where they run. Never a
`dev`/`prod` profile.

| Profile | Purpose |
|---|---|
| `qulice` | static analysis (Checkstyle+PMD+ErrorProne+EO rules) |
| `jacoco` | coverage instrumentation + gates |
| `pitest` | mutation testing gate for libraries |
| `revapi` | public-API compatibility gate |
| `deep` | slow/deep tests (tag- or module-scoped) |
| `sonatype` | Maven Central deploy (signing + `deploy`) |
| `site` | documentation site (`site`, `site-deploy`) |
| `rultor`/`<product>` | external deploy profile from remote settings |

**[SHOULD]** Transient "skip" flags are properties (`-DskipTests`, `-DskipITs`), not
profiles; deactivate an inherited execution in one module with `<phase>none</phase>`, never by
editing the parent.

## 8. Reproducible builds

**[MUST]** Commit the Maven wrapper (`mvnw`, `.mvn/wrapper/`), pin UTF-8 everywhere and
normalize line endings in `.gitattributes`:

```
# .mvn/jvm.config — deterministic CI heap, no surprise OOM
-Xmx4096m
-Xms1024m
-XX:+HeapDumpOnOutOfMemoryError
# .gitattributes
* text=auto eol=lf
*.java ident
*.xml ident
```

**[SHOULD]** Pin every plugin version (parent or explicit); no `LATEST`/ranges. Run the build
in a disposable container, non-root inside it (`Adt.33`, `Adt.34`).

## 9. argLine composition for coverage

**[MUST]** Never hard-code `argLine`. Declare an empty `argLine` property and *append* to it,
so instrumentation agents (Jacoco) inject without being overwritten (see §12):

```xml
<properties>
  <argLine/>   <!-- empty by default; agents append via @{argLine} -->
</properties>
```

**[MUST]** Expose JVM sizing as properties (`heap-size`, `stack-size`) threaded into
surefire/failsafe; large CI jobs override with `-Dheap-size=24G`. **[SHOULD]** Keep tests
deterministic: `-Duser.language=en -Duser.country=US -Djava.awt.headless=true`.

## 10. Unit vs integration separation

**[MUST]** Two phases, two plugins, two naming conventions (config in §12):

| | Unit | Integration |
|---|---|---|
| Plugin | `maven-surefire-plugin` | `maven-failsafe-plugin` |
| Pattern | `*Test.java` | `*ITCase.java` / `*IT.java` |
| Phase | `test` | `verify` (fails the build) |
| Speed | fast, every commit | slow, CI / real deps |

**[MUST]** Integration tests run against real infrastructure via profiles (DynamoDBLocal, H2,
a real server on a reserved port), never mocks (`Adt.40`, `Adt.41`). **[SHOULD]** Keep ITs in
a dedicated module with `<runOrder>random</runOrder>` to surface order dependence; select
suites with tags (`slow`, `deep`, `snippets`), not duplicated config.

## 11. Build tiers

**[SHOULD]** Four tiers, each optimized for its consumer: 1) **Local fast** — `mvn test` (no
Qulice, exclude `slow`); 2) **PR gate** — `mvn --errors --batch-mode clean install -Pqulice`,
OS×JDK matrix; 3) **Preflight merge** — the bot re-runs the PR gate on top of `master`;
4) **Release** — `versions:set` + `-Pqulice -Psonatype` signed deploy, tag-driven.

---

## 12. `pom.xml` skeleton (well-commented)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!-- SPDX-FileCopyrightText: Copyright (c) 2009-2026 Yegor Bugayenko -->
<!-- SPDX-License-Identifier: MIT -->
<project xmlns="http://maven.apache.org/POM/4.0.0" ...>
  <modelVersion>4.0.0</modelVersion>

  <!-- Policy parent: plugin versions, JDK config, license, Qulice wiring. -->
  <parent>
    <groupId>com.jcabi</groupId>
    <artifactId>parent</artifactId>
    <version>0.73.4</version>
  </parent>

  <groupId>io.example</groupId>
  <artifactId>example</artifactId>
  <version>1.0-SNAPSHOT</version>            <!-- released only by the tag bot -->
  <packaging>jar</packaging>

  <properties>
    <maven.compiler.release>17</maven.compiler.release>   <!-- == CI JDK == docs -->
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
    <argLine/>                                <!-- agents append via @{argLine} -->
    <heap-size>1024m</heap-size>
    <stack-size>512M</stack-size>
    <jacoco.line>0.65</jacoco.line>
    <pitest.mutation>75</pitest.mutation>
  </properties>

  <dependencyManagement>
    <dependencies>
      <dependency>
        <groupId>org.junit</groupId>
        <artifactId>junit-bom</artifactId>
        <version>6.1.3</version>
        <type>pom</type><scope>import</scope>
      </dependency>
      <!-- pin domain deps here; children omit <version> -->
    </dependencies>
  </dependencyManagement>

  <dependencies>
    <!-- library runtime: keep tiny; ideally zero non-provided deps -->
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

  <build>
    <plugins>
      <!-- Toolchain floor: fail fast on the wrong JDK -->
      <plugin>
        <artifactId>maven-enforcer-plugin</artifactId>
        <executions><execution>
          <id>enforce-java</id><phase>validate</phase>
          <goals><goal>enforce</goal></goals>
          <configuration><rules>
            <requireJavaVersion><version>[17,)</version></requireJavaVersion>
          </rules></configuration>
        </execution></executions>
      </plugin>
      <!-- Compiler: release is the single source of truth -->
      <plugin>
        <artifactId>maven-compiler-plugin</artifactId>
        <configuration><release>${maven.compiler.release}</release></configuration>
      </plugin>
      <!-- Unit tests; @{argLine} keeps Jacoco injection working -->
      <plugin>
        <artifactId>maven-surefire-plugin</artifactId>
        <configuration>
          <argLine>@{argLine} -Dfile.encoding=UTF-8 -Xmx${heap-size} -Xss${stack-size}</argLine>
          <excludedGroups>slow</excludedGroups>
        </configuration>
      </plugin>
      <!-- Integration tests: fail the build at verify -->
      <plugin>
        <artifactId>maven-failsafe-plugin</artifactId>
        <configuration><forkCount>1.0C</forkCount><reuseForks>true</reuseForks></configuration>
      </plugin>
      <!-- License header check on every file at verify -->
      <plugin>
        <groupId>com.mycila</groupId>
        <artifactId>license-maven-plugin</artifactId>
        <executions><execution>
          <phase>verify</phase><goals><goal>check-file-header</goal></goals>
        </execution></executions>
      </plugin>
    </plugins>
  </build>

  <profiles>
    <!-- Static analysis: mandatory in CI, opt-in locally -->
    <profile>
      <id>qulice</id>
      <build><plugins><plugin>
        <groupId>com.qulice</groupId>
        <artifactId>qulice-maven-plugin</artifactId>
        <configuration>
          <!-- bundle-only rule set; each exclusion needs a linked reason -->
          <excludes><exclude>duplicatefinder:.*</exclude></excludes>
        </configuration>
        <executions><execution><goals><goal>check</goal></goals></execution></executions>
      </plugin></plugins></build>
    </profile>
    <!-- Coverage gate: fail below the threshold -->
    <profile>
      <id>jacoco</id>
      <build><plugins><plugin>
        <artifactId>jacoco-maven-plugin</artifactId>
        <executions>
          <execution>
            <goals><goal>prepare-agent</goal><goal>report</goal></goals>
          </execution>
          <execution>
            <id>check</id><phase>verify</phase><goals><goal>check</goal></goals>
            <configuration><rules><rule>
              <element>BUNDLE</element>
              <limits><limit>
                <counter>LINE</counter><value>COVEREDRATIO</value>
                <minimum>${jacoco.line}</minimum>
              </limit></limits>
            </rule></rules></configuration>
          </execution>
        </executions>
      </plugin></plugins></build>
    </profile>
    <!-- Mutation gate for libraries -->
    <profile>
      <id>pitest</id>
      <build><plugins><plugin>
        <groupId>org.pitest</groupId>
        <artifactId>pitest-maven-plugin</artifactId>
        <configuration><mutationThreshold>${pitest.mutation}</mutationThreshold></configuration>
        <executions><execution><goals><goal>mutationCoverage</goal></goals></execution></executions>
      </plugin></plugins></build>
    </profile>
    <!-- Public-API compatibility: every accepted break needs a justification -->
    <profile>
      <id>revapi</id>
      <build><plugins><plugin>
        <groupId>org.revapi</groupId>
        <artifactId>revapi-maven-plugin</artifactId>
        <executions><execution><goals><goal>check</goal></goals></execution></executions>
      </plugin></plugins></build>
    </profile>
    <!-- Slow/deep tests: CI only -->
    <profile><id>deep</id><properties><groups>slow</groups></properties></profile>
    <!-- Signed Maven Central deploy (tag-driven only) -->
    <profile>
      <id>sonatype</id>
      <build><plugins><plugin>
        <artifactId>maven-gpg-plugin</artifactId>
        <executions><execution>
          <phase>deploy</phase><goals><goal>sign</goal></goals>
        </execution></executions>
      </plugin></plugins></build>
    </profile>
    <!-- Documentation site -->
    <profile><id>site</id></profile>
  </profiles>
</project>
```

---

## 13. Checklist

- [ ] One command, documented: `mvn --errors --batch-mode clean install -Pqulice`.
- [ ] `com.jcabi:parent` inherited; repo POM thin.
- [ ] `maven.compiler.release` == CI JDK == README JDK; enforcer floor present.
- [ ] Version `-SNAPSHOT`; release via `versions:set` from a validated semver tag.
- [ ] All versions in `dependencyManagement`/BOM; children omit versions.
- [ ] Zero non-`provided` runtime deps for a library; exclusions flatten the graph.
- [ ] Profiles named by purpose; no environment-named profiles.
- [ ] Wrapper + `.mvn/jvm.config` + UTF-8 committed.
- [ ] `argLine` starts empty; JVM sizing via properties.
- [ ] `*Test` surefire / `*ITCase` failsafe; `verify` fails the build.
- [ ] Qulice, Jacoco and PIT gates in the POM, run by the single command.

---

## 14. Running long builds safely (agent guidance)

Dependency resolution over a slow or legacy repository can look like a hang and stall an agent
forever. Never run a bare, unbounded Maven command in the foreground.

- **[MUST]** Always pass `--batch-mode` (and `-q` where noise is unwanted); never run Maven in a
  mode that waits for interactive input.
- **[MUST]** Bound the command with a timeout; a dependency download can block with no output.
- **[MUST]** Do **not** pipe a long build into a buffering consumer (`Select-Object -Last`,
  `head`, `tail`): it withholds all output until the process ends, so the build *looks* hung.
  Redirect to a log file and poll the tail instead.
- **[MUST]** Prefer Maven Central: legacy repos such as `oss.sonatype.org` (pulled in by some
  parents) are slow and can stall. Force a mirror in `settings.xml`:
  ```xml
  <settings><mirrors><mirror>
    <id>central</id><mirrorOf>*</mirrorOf>
    <url>https://repo1.maven.org/maven2</url>
  </mirror></mirrors></settings>
  ```
  then run `mvn -s settings.xml …`.
- **[SHOULD]** Run heavy builds in the background writing to a log, and poll the process + tail:
  ```bash
  mvn --batch-mode clean install -Pqulice > build.log 2>&1 &
  # poll: tail -n 5 build.log ; check the process is still alive
  ```
- **[SHOULD]** Reuse the local repository (`~/.m2`): after the first resolve, builds are fast.
  When the network is unavailable, go offline with `-o`.
- **[SHOULD]** If a build makes no progress for several minutes, kill it cleanly and retry — a
  partial download is simply re-fetched.
- **[NICE]** `-Dstyle.color=never` keeps logs clean; on Windows quote `-D` args (`"-DskipTests"`).

### Anti-hang checklist

```text
[ ] --batch-mode, no interactive prompts
[ ] timeout set on the command
[ ] output to a log file, poll the tail (never Select-Object -Last)
[ ] Central mirror via -s settings.xml (avoid slow legacy repos)
[ ] background + alive-check for very long runs
[ ] -o offline when deps are cached
```
