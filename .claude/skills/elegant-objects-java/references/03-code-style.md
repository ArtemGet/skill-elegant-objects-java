# 03 — Code style & quality gate

Reference for an AI agent writing Java in Elegant Objects style. This file is the **enforced**
style contract: every rule here is checked by Qulice (Checkstyle + PMD + ErrorProne + EO-specific
checks). Do not hand-format; write code that passes the gate on the first run.

Priority legend: **[MUST]** build fails without it · **[SHOULD]** strong default, deviate only with
a written reason · **[NICE]** improves elegance when free. If you are uncertain whether something
is a style issue, it is one — assume a stranger reads this at 3 a.m. and pick the simpler form.

## 1. Formatting & layout

### Rules

- **[MUST] LF line endings only**, and the file **ends in a newline** (`NewlineAtEndOfFile lineSeparator=lf`, `UnixEndOfLine`).
- **[MUST] Lines ≤ 100 columns.** Exempt: import lines, Javadoc URLs, string continuations.
- **[MUST] File length ≤ 1000 lines.** Split the class instead of requesting an exclusion.
- **[MUST] No tabs.** Indent with spaces only.
- **[MUST] No trailing whitespace** on any line.
- **[MUST] No two consecutive blank lines**, and **no blank line before a closing brace**.
- **[MUST] No blank line inside a method body.** A blank line means "extract a method or class."
- **[MUST] Paired brackets**: every `{`/`(`/`[` closes correctly; the closing bracket of a
  multi-line block sits on its own line.
- **[MUST] Monotonic indentation**: indentation rises and falls with nesting, never oscillating.
- **[MUST] No line ends with an operator** (`. - + % / * < >`, except `->`); **no line starts
  with `=`**.
- **[MUST] Wrapping**: `.`, `@`, `::` lead the continued line; `,`, `;`, `...` stay at line end;
  one space separator; braces/whitespace around tokens.
- **[MUST] A fluent multi-line call stays attached**; the leading `.` opens the continuation.

### Smells → fix

| Smell | Fix |
|---|---|
| CRLF check-in | `.gitattributes` `* text=auto eol=lf`; save as LF |
| 118-char line | extract a local object/variable; never split with `+` |
| Blank line mid-method | extract the block into a named method |
| dot at end of line, call below | move the dot to the start of the continued line |
| `{` alone after `if (x)` | K&R: `if (x) {` |
| 1400-line class | split by responsibility; classes < 250 LOC |

```java
final String text = new UncheckedText(
    new TextOf(this.file)
).asString();
```

## 2. Imports

### Rules

- **[MUST] No star imports** (`import java.util.*;`).
- **[MUST] No static imports** (`import static ...`).
- **[MUST] No redundant imports** (same package, `java.lang`, unused) and **no illegal imports**.
- **[MUST] Imports ordered and cohesive**; never wrap an import across lines.
- **[SHOULD] Fully unqualified** in statements: no `java.lang.` prefix, no fully-qualified type
  names inline (`UnnecessaryJavaLangCheck`, `FullyQualifiedTypeCheck`).

### Smells → fix

| Smell | Fix |
|---|---|
| `import java.util.*;` | import each class explicitly |
| `import static ...Matchers.*;` | `import org.hamcrest.Matchers;` + qualify |
| `new java.util.ArrayList<>()` | import `ArrayList`, then `new ArrayList<>()` |
| imports out of order | alphabetical, grouped by root package |

```java
import java.util.ArrayList;
import java.util.List;
import org.hamcrest.MatcherAssert;
```

## 3. Immutability, `final`, and `this.`

### Rules

- **[MUST] Classes are `final`** (or `abstract` for an envelope base) — `FinalClass`, `ProhibitNonFinalClassesCheck`.
- **[MUST] Every field is `private final`** — `VisibilityModifier`, `FinalLocalVariable`.
- **[MUST] Parameters and local variables are `final`** — `FinalParameters`, `FinalLocalVariable`.
- **[MUST] Field access prefixed with `this.`** (`RequireThis`); never `this.STATIC`.
- **[MUST] No parameter reassignment** (`ParameterAssignment`).
- **[MUST] No setters; mutation returns a new object** (`with(...)` / constructor).
- **[MUST] Defensive copies** on array/collection boundaries; never store or return a live array
  (`ArrayIsStoredDirectly`).
- **[SHOULD] Return unmodifiable collections** at public boundaries.

### Smells → fix

| Smell | Fix |
|---|---|
| `class Foo {` | `final class Foo {` |
| `private int x;` | `private final int x;` |
| `void mul(int f)` | `void mul(final int f)` |
| `x = 0;` inside body | make it `final` at declaration |
| `name` for a field | `this.name` |
| `this.STATIC` | `Type.STATIC` |
| setter `setX(v)` | return `new Foo(...)` / `withX(v)` |

```java
final class Cash {
    private final int dollars;
    Cash(final int amount) {
        this.dollars = amount;
    }
    Cash mul(final int factor) {
        return new Cash(this.dollars * factor);
    }
}
```

## 4. Size, complexity & the single return

### Rules

- **[MUST] Exactly one `return` per method** (`ReturnCount max=1`). Rewrite with a single
  expression or a Null Object — **never** add an exclusion.
- **[MUST] ≤ 3 parameters** (`ParameterNumberCheck max=3`). More arguments → extract a collaborator.
- **[MUST] ≤ 40 executable statements per method**; lambda body ≤ 20.
- **[MUST] Bounded complexity**: cyclomatic/NPath/boolean-complexity within limits; class fan-out
  ≤ 30; limited nesting for `for`/`if`/`try`/`switch`.
- **[SHOULD] < 5 public/protected methods and < 250 LOC per class**; extract at > 7 methods.
- **[SHOULD] No one-use private constants** (inline them) and no one-time local variables.

### Smells → fix

| Smell | Fix |
|---|---|
| `if (x) return a; ... return b;` | single `return` guarded by the condition |
| two `return` statements | introduce a local result and return once |
| `void f(a, b, c, d)` | extract a parameter object / collaborator |
| 60-line method | split into named private methods |
| `private static final int ONE = 1;` used once | inline `1` |

```java
int fee(final int age) {
    return age < 18 ? 0 : 10;
}
```

## 5. Tokens & statements

### Rules

- **[MUST] No `++` or `--`** (`IllegalToken POST_INC,POST_DEC`). Use `x = x + 1`.
- **[MUST] No inline conditional expressions** where the check forbids them; prefer extraction.
- **[MUST] No `clone()` and no `finalize()`** (`NoClone`, `NoFinalizer`, `SuperClone`, `SuperFinalize`).
- **[MUST] No C-style single-line comments**; comments start with a capital letter.
- **[MUST] No comments inside a method body** (`MethodBodyCommentsCheck`) — names carry meaning.
- **[MUST] No `instanceof`/casting/reflection** in business logic; the only tolerated runtime type
  test is `equals(Object)`.
- **[MUST] No shared public constants/enums**; use micro-classes.
- **[SHOULD] No string `+` for messages**; use `String.format` with indexes.
- **[SHOULD] Simple beats clever**; assume a junior reader.

### Smells → fix

| Smell | Fix |
|---|---|
| `i++;` | `i = i + 1;` |
| `if (a) x = b;` inline | braces, or extract |
| `obj.clone()` | copy via a constructor / `with(...)` |
| `// increment counter` | delete; name the variable `count` |
| `instanceof Foo` | polymorphism / decorator |
| `"a" + b + "c"` in a message | `String.format("a %s c", b)` |

## 6. Constructors (style-enforced subset)

- **[MUST] Constructors are code-free**: only assignment or `this(...)`/`super(...)`. No method
  calls, regex, parsing, `String.format`, or `new` of collaborators (`ConstructorsCodeFreeCheck`,
  `ConstructorOnlyInitializesOrCallOtherConstructors`).
- **[MUST] Exactly one primary constructor, declared last** (`ConstructorsOrderCheck`).
- **[MUST] Implicit constructors declared explicitly** (`ImplicitConstructorCheck`).
- **[MUST] Only one constructor performs initialization** (`OnlyOneConstructorShouldDoInitialization`).
- **[MUST] No field initializers when a constructor exists** (`ConstructorShouldDoInitialization`).

### Smells → fix

| Smell | Fix |
|---|---|
| `this.name = name.trim();` | `this(new Trimmed(name))` |
| `this.list = list` (live array) | `this.list = Arrays.copyOf(list, list.length)` |
| logic in the primary ctor | move to a method; ctor only assigns |
| ctor declared before secondaries | primary ctor goes **last** |

```java
final class Name {
    private final String text;
    Name(final String raw) {
        this(new Trimmed(raw));
    }
    Name(final Trimmed txt) {
        this.text = txt.toString();
    }
}
```

## 7. Naming

### Rules

- **[MUST] Class name = what the object *is***; no `-er`/`-or` job titles (`FileReader` → `DataFile`).
- **[MUST] No `get`/`set` prefixes**; name a returning method for what it returns.
- **[MUST] Field/local `^(id|[a-z]{3,12})$`; parameters `^(id|[a-z]{3,})$`; catch params `^(ex|[a-z]{3,12})$`.
- **[MUST] Method names** `^(as|at|by|go|id|in|is|it|of|on|or|to|up|[a-z]{2,}[a-zA-Z]+)$`; verbs for
  manipulators (`void`), nouns for builders (return a value), never both.
- **[MUST] Only `IT` is an allowed abbreviation** (`AbbreviationAsWordInName`).
- **[MUST] `package-info.java`** with a one-line Javadoc in every package.
- **[SHOULD] One-word noun names**; avoid compounds. **No `*Client` suffix.**
- **[SHOULD] Test classes `final`, package-private, `FooTest`/`FooITCase`**; no `test` prefix.
- **[NICE] Entity prefixes on implementations** (`Rq*`, `Rs*`, `Tk*`, `Default*`, `Mk*`).

### Smells → fix

| Smell | Fix |
|---|---|
| `class UserValidator` | a noun entity, e.g. `VaidUser` |
| `getName()` | `name()` |
| `int n;` (1 char) | `int num;` |
| `ArrayList lst` | `List items` |
| `class FooTest` not final | `final class FooTest` |
| `void testTotal()` | `void totalsAmount()` |

## 8. Javadoc & comments

### Rules

- **[MUST] Every type, public method, and field has Javadoc** (`JavadocType`, `JavadocMethod`,
  `JavadocVariable`, `MissingJavadocType/Package`).
- **[MUST] Full tags in fixed order**: `@param`, `@return`, `@throws`, `@since`. Every public method
  carries `@since`; `@param` order matches the signature (`JavadocParameterOrderCheck`).
- **[MUST] Tag descriptions start with a capital letter** and end with a period.
- **[MUST] No Javadoc for overridden or private methods** (`NoJavadocForOverriddenMethods`,
  `NoJavadocForPrivateMethods`).
- **[MUST] No inline narration inside methods** — no `//` comments explaining internals.
- **[SHOULD] Document external interfaces only**; names and structure replace internal docs.

Tension resolved: impose **Javadoc on the public contract** and **ban inline narration**. A
`// loop over the list` comment is always deleted; `@param name The user name` is required.

### Smells → fix

| Smell | Fix |
|---|---|
| public method, no Javadoc | add summary + tags + `@since` |
| `@return result` no capital | `@return the computed total` |
| `// parse the input` in a method | delete; extract `new Parsed(input)` |
| `@param` in wrong order | reorder to match the signature |
| Javadoc on a private helper | delete it |

```java
/**
 * Sums two amounts.
 * @param left The left amount
 * @param right The right amount
 * @return The total
 * @since 0.1
 */
int total(final int left, final int right) {
    return left + right;
}
```

## 9. The Qulice rule set

Qulice is a Maven plugin bundling three analyzers behind one `verify`-phase goal, `check`:
Checkstyle 14.1.0 (layout/imports/Javadoc/naming/design, `checks.xml`); PMD 7.26/7.28 (Java
categories + ~15 custom `com.qulice.pmd.rules`, `ruleset.xml`); ErrorProne 2.50 (default
bug-pattern set, forked `javac`); plus 5 sequential Maven validators (pom-xpath, enforcer,
dependency-analyzer, duplicate-finder, snapshots).

- **[MUST] Run Qulice in the build**, bound to `verify`; any `Violation` fails it.
- **[MUST] Bundle and lock the rule set.** Consumers may only **exclude** files/checkers or append
  `-Xep:` flags — **never redefine** a rule.
- **[MUST] Actionable violations**: `validator: file[line]: message (RuleName)`.
- **[MUST] Exactly-one-assert tests, `final` test classes, no plain JUnit asserts.**
- **[MUST] Never leave a suppression covering nothing** (`UnusedSuppressions`).
- **[MUST] When two analyzers contradict, disable one rule with a written rationale.**
- **[SHOULD] Lint test code too** (`jtcop`), enforce architecture (`ArchUnit`) on large projects.

### Most important EO-specific Qulice checks

| Check | Forbids | Replacement |
|---|---|---|
| `ConstructorsCodeFreeCheck` | any code in ctors | assignments / `this(...)` / wrap args |
| `ProhibitPublicStaticMethods` | public `static` methods | constructor-injected objects (only `main`) |
| `AvoidDirectAccessToStaticFields` | bare `FIELD` | `ClassName.FIELD` (or remove the static) |
| `AvoidAccessToStaticMembersViaThis` | `this.STATIC` | `Type.STATIC` / object |
| `StaticAccessViaInstanceCheck` | static access via an instance | direct class reference |
| `NonStaticMethodCheck` | non-static method not using `this` | make it `static` or move state in |
| `ProhibitStaticNestedClassesCheck` | static nested classes | top-level `final` class |
| `FinalClass` / `ProhibitNonFinalClassesCheck` | non-final classes | `final` (or `abstract` base) |
| `ProtectedMethodInFinalClassCheck` | `protected` in a final class | `private` |
| `RedundantSuperConstructorCheck` | needless `super()` | delete |
| `ReturnCount=1` | multiple returns | single expression / Null Object |
| `ParameterNumberCheck` | > 3 params | extract collaborator |
| `SingleUseConstantCheck` | constant used once | inline it |
| `JUnitTestClassShouldBeFinal` | non-final test class | `final class ...Test` |
| `UnitTestContainsTooManyAsserts` | ≥ 2 asserts/test | one `assertThat` |
| `ProhibitPlainJunitAssertionsRule` | `Assert.assert*` | Hamcrest `MatcherAssert.assertThat` |
| `ProhibitFieldsInTestClassesCheck` | fields in tests | literals in each test |
| `ConstructorShouldDoInitialization` | field initializers + ctor | init inside the one ctor |
| `OnlyOneConstructorShouldDoInitialization` | two ctors doing work | funnel to the primary |
| `ImplicitConstructorCheck` | implicit ctor | declare it explicitly |
| `UnnecessaryJavaLangCheck` | `java.lang.` prefix | import + bare name |
| `FullyQualifiedTypeCheck` | fully-qualified name inline | import + bare name |
| `UnusedPackagePrivateClasses` | package-private type nobody uses | delete or make public |
| `MethodBodyCommentsCheck` | comments inside methods | self-documenting names |

Full rule sets live in `knowledge/elegant-objects-java.md` §F.

## 10. Quality-gate recipe & build commands

### Rules

- **[MUST] Bind the plugin to `verify`** (goal `check`) with the project license file.
- **[MUST] Keep the gate in a profile** (`qulice`) so `mvn install` stays fast; CI/release run it.
- **[MUST] One non-interactive build command**, documented in the README.
- **[SHOULD] Gate coverage (Jacoco), mutation (PIT), and API compatibility (Revapi)** in the POM;
  ~0.65 line / 75% mutation is a realistic binding floor. Every Revapi break needs a written
  `<justification>`, never a silent acceptance.

### Ready-to-paste plugin snippet

```xml
<profiles>
  <profile>
    <id>qulice</id>
    <build>
      <plugins>
        <plugin>
          <groupId>com.qulice</groupId>
          <artifactId>qulice-maven-plugin</artifactId>
          <version>0.35.1</version>
          <configuration>
            <license>file:${basedir}/LICENSE.txt</license>
          </configuration>
          <executions>
            <execution>
              <goals><goal>check</goal></goals>
            </execution>
          </executions>
        </plugin>
      </plugins>
    </build>
  </profile>
</profiles>
```

### Exact build commands

```bash
mvn --errors --batch-mode clean install -Pqulice        # local: build + gate + tests
mvn -B --no-transfer-progress verify -Pqulice -DskipTests -DskipITs   # CI: gate only
```

### Coverage gate (POM shape)

Bind `jacoco-maven-plugin` goals `prepare-agent` and `check`; in `check` add a `BUNDLE` rule with a
`LINE` limit (e.g. `0.65`). Use an empty `<argLine/>` property plus `@{argLine}` so the agent can
inject JVM flags. Add `revapi-maven-plugin` for binary compatibility and `pitest-maven-plugin`
(mutation, ~75% floor) for libraries.

## 11. Handling a violation

Never blanket-suppress. Follow this ladder, in order:

1. **Fix the code.** 95% of violations are a real smell; this is the expected outcome.
2. **Narrow the exclusion** if the rule conflicts with a legitimate EO construct — smallest scope
   (one file/path/rule) in the plugin config:
   ```xml
   <configuration>
     <excludes>
       <exclude>checkstyle:.*/generated/.*</exclude>
       <exclude>pmd:.*/model/.*</exclude>
     </excludes>
   </configuration>
   ```
3. **Disable one analyzer rule** only when two analyzers contradict, with a written rationale
   (ErrorProne via appended `<errorprone><flag>-Xep:Rule:OFF</flag></errorprone>`).
4. **Inline suppression** for a genuine one-off: `@checkstyle Name (N lines)` or
   `@SuppressWarnings("Name")`. It **must** name a real check and cover exactly the offending lines.
5. **Attach a ticket** (`#123`) and rationale to every exclusion and suppression.

### Rules

- **[MUST] Never `@SuppressWarnings` an entire class/method** to silence many rules.
- **[MUST] An unused suppression is itself a build failure.**
- **[MUST] Every deviation has a written reason and a ticket reference.**
- **[SHOULD] Prefer a code fix over any exclusion**, every time.

### Smells → fix

| Smell | Fix |
|---|---|
| `@SuppressWarnings("all")` on a class | fix each rule; exclude per-rule only if truly needed |
| `@checkstyle ReturnCount (50 lines)` | rewrite to a single return |
| exclusion for `.*` | narrow to the exact package/rule |
| suppression with no ticket | add `#123` + rationale, or remove it |
| disabled rule with no rationale | document it in the ruleset or re-enable |

## 12. Style checklist (a Java file that passes Qulice)

- [ ] LF endings, no tabs, no trailing spaces, ≤100-col lines, ends in a newline.
- [ ] No blank line inside a method; none before a closing brace; no double blank lines.
- [ ] File ≤ 1000 lines; class ≤ 250 LOC and ≤ 5 public/protected methods.
- [ ] `package-info.java` exists; every type/public method/field has Javadoc.
- [ ] Javadoc tags in order (`@param`→`@return`→`@throws`→`@since`), capitalised, with periods.
- [ ] No comments inside method bodies; no `//` narration.
- [ ] Class is `final` (or `abstract`); fields `private final`; params and locals `final`.
- [ ] `this.` on every field; no `this.STATIC`; no bare static access.
- [ ] Constructors only assign / delegate; exactly one primary ctor, declared last.
- [ ] Exactly one `return`; ≤3 parameters; ≤40 statements; bounded complexity/nesting.
- [ ] No `++`/`--`, no inline conditionals, no `clone()`/`finalize()`.
- [ ] No star/static/redundant imports; imports ordered, cohesive, never wrapped.
- [ ] Names: nouns for classes, no `-er`, no `get`/`set`, only `IT` abbreviated.
- [ ] No `instanceof`/casting/reflection in business logic.
- [ ] Tests: `final` class, no fields, exactly one Hamcrest `assertThat`, no `Assert.assert*`.
- [ ] No suppression that covers nothing; every suppression names a check + ticket.
- [ ] `mvn --errors --batch-mode clean install -Pqulice` passes locally before commit.

---

## 13. Upgrading Qulice (new rules surface)

A Qulice version bump is a **separate task** — its own issue/PR. Newer releases add and tighten
rules, so the build fails with violations the old version tolerated.

- **[MUST]** Fix every violation; do **not** add blanket `@SuppressWarnings`/`@checkstyle` or
  exclude files/rules to go green. A narrow, commented, ticket-linked suppression is a last resort.
- **[MUST]** `ConstructorsCodeFreeCheck`: a constructor must not call methods. Replace
  `Collections.unmodifiable*` / `List.copyOf(...)` / `Arrays.asList(...)` with **constructor-based**
  equivalents (`new ArrayList<>(...)`, `new LinkedList<>(...)`, a Cactoos `new ListOf<>(...)`), or
  move preprocessing into a secondary `this(...)` ctor — never leave a method call in the ctor.
- **[MUST]** `EmptyLineBeforeFirstMemberCheck`: a blank line after every type-opening brace.
- **[MUST]** `ImplicitConstructorCheck` / `MissingJavadocMethodCheck`: declare an explicit ctor
  with Javadoc; document every public ctor.
- **[MUST]** `JavadocUnclosedParagraphCheck`: close `<p>` with `</p>` (e.g. `<p><img …></p>`).
- **[MUST]** `JavadocTagsDotCheck`: no trailing dot in `@param`/`@return` text.
- **[MUST]** `SingleUseConstantCheck`: inline a private constant used only once.
- **[MUST]** `IllegalCatchCheck` / `AvoidCatchingGenericException`: never `catch (Exception)`;
  in tests use `Assertions.assertThrows(SpecificException.class, …)`.
- **[SHOULD]** `@FunctionalInterface` on every single-abstract-method interface.
- **[SHOULD]** Test `equals(null)` / a foreign type without tripping PMD `EqualsNull`:
  ```java
  assertThat("not null", new Thing(), not(equalTo(null)));
  assertThat("not foreign", new Thing(), not(equalTo("x")));
  ```
  This still calls `Thing.equals(null)` via the matcher — keep the assertion meaningful. Do **not**
  weaken it to `assertNotEquals(thing, null)`, which only checks the object is non-null.
- **[MUST]** After the upgrade, re-run the whole build and confirm the quality gates still pass.
- **[MUST]** Trust CI over a local Windows run for ErrorProne-affected rules. Qulice 0.36's
  ErrorProne integration can fail to decode its forked output on Windows
  (`java.nio.charset.MalformedInputException`) and silently **drop** findings, so a Windows-local
  `-Pqulice` build can be green while Linux CI fails on a real violation (observed: ErrorProne
  `AvoidCommonTypeNames` for a nested class shadowing `java.lang.Void`). When CI is red but local is
  green, fetch the CI job log (GitHub MCP `get_job_logs`) and fix what it reports — do not trust the
  local green. Renaming the clashing type (e.g. `Void` → `Empty`) is the fix; do not suppress.
