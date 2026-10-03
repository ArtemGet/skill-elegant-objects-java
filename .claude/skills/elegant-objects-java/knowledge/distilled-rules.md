# Elegant Objects Java — distilled rulebook

Drafting source for the skill's `SKILL.md` + reference docs. Distilled from the book postulates
(`B*`), the article additions (`A*`), and ten real yegor256-style repositories (Cactoos, Takes,
Qulice, eo, s3auth, xembly, rultor, jare, requs, rehttp). Deduplicated; where sources disagree the
default is stated first, followed by the exception.

Priority legend: **[MUST]** non-negotiable / the build should fail on it · **[SHOULD]** strong default,
deviate only with a written reason · **[NICE]** improves elegance, judged case by case.

Evidence points into `knowledge/elegant-objects-java.md` by id (`B1.1`, `Acs.1`, …) and into the code analyses by
file/line, so a skill author can expand any rule into a tutorial or a static check.

---

## 0. EO in one page (orientation)

**What EO is:** an object-oriented *paradigm* (not a library) that renounces classic techniques. A
class is where objects are born — not a template of functions, not a bag of statics. Objects are
living representatives of real-world entities.

**The 11 principles (enforced where marked):**
1. No `null` — use Null Objects / exceptions / collections. *(enforced: `NoNulls` + Qulice)*
2. No code in constructors — ctors only assign. *(enforced: `ConstructorsCodeFreeCheck`)*
3. No getters/setters — tell, don't ask.
4. No mutable objects — all fields `private final`. *(enforced: `FinalParameters`/`FinalLocalVariable`)*
5. No `-er` names — name what the object *is*.
6. No static methods, not even private — statics are global state. *(enforced:
   `ProhibitPublicStaticMethods`)*
7. No `instanceof`/casting/reflection — use polymorphism. *(enforced: `IllegalType`/review)*
8. No public method without a contract (interface). *(enforced: design-for-extension)*
9. No statements in tests except one `assertThat`. *(enforced: `UnitTestContainsTooManyAsserts`)*
10. No ORM/ActiveRecord — SQL is a detail behind objects.
11. No implementation inheritance — `final` classes + decoration. *(enforced: `FinalClass`)*

**The 7 virtues:** exists in real life · works by contracts · is unique (encapsulates) · is immutable ·
has nothing static · name is not a job title · is `final` or `abstract`.

**The 4 life phases (book):** Birth → Education → Employment → Retirement (i.e. design, name/size,
behavior/errors, release/maintenance). The practical craft lives in the first three; §9–§11 of
`knowledge/elegant-objects-java.md` add build/CI/repo practice the book does not cover.

**The top anti-patterns:** NULL, utility classes, mutable objects, getters/setters, DTO, ORM,
singletons, `-er` controllers/managers/validators, public statics, class casting, traits/mixins,
builders, DI containers, behavior-carrying annotations.

**Definition of Done for a change:** one small PR tied to a ticket · tests pass (one assertion each,
no fixtures) · Qulice green · coverage/mutation above the gate · a reproducing test for any bug ·
commit references `#ticket` · docs/version synced · release is one command.

**The build command:** `mvn --errors --batch-mode clean install -Pqulice`.

---

## Top 20 non-negotiable rules

1. **[MUST] No `null` across any public contract.** Never accept, return or signal with `null`; use a
   Null Object, a collection, or throw. Centralize unavoidable JDK-boundary checks in a decorator
   (`B6.1`–`B6.3`, `NoNulls.java:31-45`).
2. **[MUST] No `public static` behavior.** No static methods in the public API, no static mutable
   state. `main`, annotation-required test providers and `private static final` constants are the only
   tolerated forms (`B1.9`, Qulice `ProhibitPublicStaticMethods`).
3. **[MUST] No utility classes and no singletons.** A bag of statics is not a class; inject
   dependencies through constructors (`B1.10`, `B1.11`).
4. **[MUST] Classes are `final` or `abstract`.** No in-between, except a documented envelope base
   whose delegate methods are `final` (`B1.4`, `B5.3`, `RsWrap.java:28`).
5. **[MUST] Every field is `private final`; objects are immutable.** Mutating operations return a new
   object (`B3.1`, `B3.2`).
6. **[MUST] Constructors only assign fields or delegate** (`this(...)`/`super(...)`). No parsing,
   validation, I/O, regex, `String.format`, or `new` of collaborators (`B2.1`, `B2.5`,
   `ConstructorsCodeFreeCheck`).
7. **[MUST] Exactly one primary constructor, declared last.** Secondary ctors funnel to it via
   `this(...)` (`B2.2`–`B2.4`).
8. **[MUST] `new` is allowed only in secondary constructors.** Methods and the primary ctor receive
   collaborators already built (`B1.19`, `B2.9`).
9. **[MUST] No getters/setters; tell, don't ask.** No `get`/`set` prefixes; expose behavior
   (`B1.13`, `B4.11`, `B4.13`).
10. **[MUST] Every public method implements an interface method.** No interface-less public API; keep
    interfaces short (`B1.8`, `B4.7`, `B4.8`).
11. **[MUST] No implementation inheritance.** Reuse by composition/decorators; only subtyping via
    interface extension and abstract-class refinement (`B5.1`, `Adt.18`).
12. **[MUST] No `-er` job-title class names.** Name what the object *is* (`B1.12`, `B4.1`, `B4.2`).
13. **[MUST] No `instanceof`, casting or reflection** in business logic; the only tolerated type test
    is the JDK `equals(Object)` contract (`B1.14`, `Acob.32`).
14. **[MUST] Fail fast with checked exceptions.** Never swallow, catch-and-log, or return sentinel
    values; chain the cause; recover once at the top (`B6.4`–`B6.10`, `B6.12`).
15. **[MUST] Tests are first-class: one statement per test.** Arrange by constructing objects, then a
    single `MatcherAssert.assertThat(reason, actual, matcher)`; no fixtures (`B7.1`–`B7.3`, `Atq.21`).
16. **[MUST] Static analysis is mandatory and fails the build.** Run Qulice (Checkstyle+PMD+
    ErrorProne+EO rules) in CI; never advisory (`B8.9`, `Atq.44`).
17. **[MUST] Coverage and mutation gates live in the POM and fail the build** (Jacoco + PIT; ~0.65
    line / 75 mutation is realistic and binding) (`Apa.26`, `Atq.50`).
18. **[MUST] Every bug fix ships with a reproducing test** (Bug-Driven Development) (`B7.12`,
    `Atq.35`).
19. **[MUST] Automated, tag-driven, one-command release.** Semver tag → `versions:set` → signed
    deploy; read-only `master`, PR-only, bot-merged (`B9.1`, `Apa.34`, `.rultor.yml`).
20. **[MUST] Every change traces to a ticket, every commit references it; history is never
    rewritten.** PDD puzzles, no force-push, no deleted comments (`Apb.30`, `Apa.9`, `Apa.107`).

---

## 1. Core OOP & design

- **[MUST]** A class is a factory of objects (`B1.1`); an object is a representative of a real-life
  entity with coordinates, not a data container (`B1.2`, `B1.3`, `Acoa.1`). If you cannot name/draw
  the entity, refactor (`Acoa.5`).
- **[SHOULD]** Prefer the smallest real entity an object can represent; a no-arg constructor that can
  do everything represents "the Universe" (`Acoa.6`, `Acoa.9`).
  ```java
  new HTTP("https://x").read();   // represents a page
  new HTTP().read("https://x");   // represents the Universe
  ```
- **[MUST]** Encapsulate state; state is identity. Override `equals`/`hashCode` from the encapsulated
  coordinates (`B1.5`, `B3.13`).
- **[SHOULD]** Encapsulate ≤4 objects (`B1.6`); a growing state is grouped into sub-objects, never a
  flat bag.
- **[MUST]** Encapsulate something at the very least; a stateless class is isomorphic to a static
  method (`B1.7`).
- **[MUST]** Prefer objects over primitives and naked data; avoid primitive obsession (`B1.16`).
- **[MUST]** Compose smaller objects into bigger ones; prefer declarative over imperative design
  (`B1.17`, `B8.6`).
  ```java
  Collection<Integer> evens = new Filtered<>(numbers, n -> n % 2 == 0);
  ```
- **[SHOULD]** Separate instantiation from execution: build the object graph, then `run()` (`B1.18`,
  `B2.12`).
- **[MUST]** Keep objects small: <5 public/protected methods and <250 LOC (Java; 100 Ruby); use these
  as refactoring triggers (`B1.20`, `B1.21`, `B8.1`, `Acob.39`). Extract at >7 methods / >4 attributes
  / >2 method args (`Acob.39`).
- **[SHOULD]** One class = one responsibility, judged by encapsulation first, size second (`Ada.3`,
  `Ada.4`). Do not apply SRP so hard that an object becomes a DTO carrier (`Ada.3`, `Ada.43`).
- **[MUST]** Keep data and its processor in the same object; no request/response DTOs (`Ada.4`,
  `Acoa.49`).
- **[SHOULD]** Decompose vertically (decorators) not horizontally (client-assembled siblings)
  (`Acob.1`).
- **[SHOULD]** Prefer composition over messaging between peers; wrap collaborators in a bigger object
  (`Ada.5`, `Ada.21`).
- **[NICE]** Many small noun-named classes are a virtue, not a smell (`Acob.2`, `Acob.49`).
- **[MUST]** Reject anti-patterns early: God object, Singleton, utility class, global mutable state,
  MVC controller (`Apa.4`, `Acob.3`, `Acoa.25`, `B12.23`). Reject Builder/Facade/Visitor/Template
  Method/mutable-Iterator (`Acoa.25`).
- **[MUST]** Reject SOLID as a design method (especially the OCP's blessing of implementation
  inheritance); use cohesion/coupling directly (`Ada.2`).
- **[SHOULD]** Treat compile-time static analysis as part of the language, not a volunteer tool
  (`Apa.2`).
- **[SHOULD]** Design a language/model where everything is an object; `byte`/`bytes` are the only
  built-ins (`Apa.1`, `Ada.7`).
- **[NICE]** Prefer aesthetics over functionality: ugly-but-working is a defect (`Apb.3`).
- **[SHOULD]** Shrink variable scope to the minimum; objects exist to make large scopes impossible
  (`Apb.1`, `Apb.2`).
- **[SHOULD]** Model absence/existence as its own quality, not merged into another (`Acob.5`).
- **[NICE]** Let the compiler infer types from the messages an object receives; prefer `var` where the
  noun carries the type (`Ada.6`, `Acs.13`).
- **[SHOULD]** Kill implicit "connectors": an object owns its data and decides, it does not pass data
  through (`B1.2`, `Acoa.2`).

## 2. Constructors & object lifecycle

- **[MUST]** Constructors are code-free: assignments or `this(...)`/`super(...)` only (`B2.1`,
  `ConstructorsCodeFreeCheck`). No method calls on arguments — wrap them (`B2.5`).
  ```java
  Cash(String s) { this(new StringAsInteger(s)); }   // wrap
  Cash(Number n) { this.dollars = n; }               // assign
  ```
- **[MUST]** Exactly one primary ctor, declared last; secondaries delegate via `this(...)` (`B2.2`,
  `B2.3`, `B2.4`).
- **[MUST]** No validation/parsing/conversion in a ctor; defer to a method (`B2.1`, `B2.7`,
  `B2.14`). Validate only representable state (null/wrong type) in the ctor; world/runtime state in
  behavior (`Acob.11`).
- **[MUST]** `new` only in secondary ctors (`B2.9`, `Acs.3`); a convenience ctor may inject sensible
  defaults (`B2.10`).
- **[SHOULD]** Prefer many constructors over many methods; 5–10 ctors is fine (`B2.6`). Use ctor
  overloading / argument maps (`B2.13`).
- **[MUST]** An immutable object is complete and solid after construction; no skeleton+setters
  (`B2.15`, `Acs.4`).
- **[SHOULD]** Defer work to methods (lazy); add a caching decorator when repeat computation matters
  (`B2.7`, `B2.8`, `B12.8`). Never cache with a mutable `null` field (`Adt.11`, `Ada.11`).
- **[MUST]** No two-step initialization: no `init()`/`setup()`/`open()` required after the ctor
  (`Acs.4`, `Acs.40`).
- **[SHOULD]** Use constructors, never static factory methods (`Adt.4`, `Acob.8`); move instance
  caching into a dedicated object (`Acob.9`).
- **[SHOULD]** Extract argument pre-processing into a prestructor class/method, not a loop in a ctor
  (`Acob.10`).
- **[SHOULD]** Own resource lifetime via `AutoCloseable` + try-with-resources; never release manually
  before every throw (`Adt.8`, `B11.7`, `Adt.72`).
- **[MUST]** Never call overridable methods from a ctor (fragile base class) (`Acs.6`).
- **[NICE]** "Decorating envelopes" (a ctor composes decorators and adds no behavior) are a legitimate
  shape (`Acob.12`).
- **[SHOULD]** Inject dependencies only through constructors — no field/setter injection, no injector
  (`Ada.8`, `Ada.10`, `Adt.10`).
- **[NICE]** Create similar objects by copying an existing one (`with`/`copy`), not by re-instantiating
  (`Acob.13`).
- **[NICE]** Let a one-man prototype be built one way, then grow it by many small increments
  (`Apa.6`).

## 3. Immutability & state

- **[MUST]** `private final` fields; no setters; mutations return a new object (`B3.1`, `B3.2`).
  ```java
  Cash mul(int f) { return new Cash(this.dollars * f); }
  ```
- **[MUST]** Immutability is an absolute project invariant, enforced by static analysis (`Apa.8`,
  `FinalParameters`, `FinalLocalVariable`, `VisibilityModifier`, `ParameterAssignment`).
- **[MUST]** No unset `null` fields; split into small cohesive classes instead (`B3.7`).
- **[SHOULD]** Immutability is "loyalty", not constancy: an immutable object may return different
  answers over time by holding only coordinates (`B3.11`, `B3.12`, `Acob.14`, `Acob.15`).
- **[MUST]** Model mutable resources (disk, network, memory) with immutable objects holding their
  coordinates (`B3.14`, `B3.15`, `Acoa.2`).
- **[SHOULD]** Return unmodifiable/immutable collections at boundaries (`Collections.unmodifiable…`,
  `ImmutableList`).
- **[MUST]** Grow by replacement: `with(...)` chains or a `Temporary.back()` interface, never
  in-place mutation (`Acob.17`).
- **[SHOULD]** Thread safety is a decorator (`Sync*`), not a concern of the core class; never trust a
  concurrent field to make a compound operation atomic. Prove thread-safety with a latch-driven test
  (`Acob.16`, `Atq.4`, `Atq.31`).
- **[MUST]** Never expose mutable data from a method; make defensive/unmodifiable copies
  (`ArrayIsStoredDirectly`).
- **[NICE]** Track `null`/`static`/mutable-class ratios as quality indicators (`Apb.6`).
- **[SHOULD]** Make objects immutable as an explicit refactoring step when inheriting foreign code
  (`Apb.5`).
- **[MUST]** Annotate public interfaces `@Immutable` so callers can trust passed-in objects
  (`Adt.9`).
- **[SHOULD]** Separate "state" (coordinates) from "behavior" so frequently changed data does not
  force mutation (`Acoa.10`).
- **[SHOULD]** Use immutability to force small, cohesive classes (a growing ctor is ugly on purpose)
  (`Acoa.12`).
- **[MUST]** Know the concrete costs of mutability you avoid: identity-mutability bugs, non-atomic
  failures, temporal coupling, side effects, thread races (`B3.3`–`B3.8`, `Acoa.13`).
- **[SHOULD]** If you must mutate in-memory data, use a `final` byte array / memory holder as the
  honest surrogate (`Acoa.15`).
- **[NICE]** Prefer inline values + a single `final` "moniker" over many reassigned variables
  (`Acs.8`).
- **[NICE]** Cache with `StickyScalar`, synchronize with `SyncScalar` (`Ada.12`).
- **[MUST]** Never expose state as naked data (even a private field + getter is naked) (`Acs.9`).

## 4. Names, interfaces & contracts

- **[MUST]** Class name = what the object *is* (`B4.1`). No `-er`/`-or` job titles (`B4.2`,
  `Acoa.16`, `Adt.12`); `FileReader` → `DataFile`, `PrimeFinder` → `PrimeNumbers`.
- **[MUST]** No `get`/`set` prefixes (`B4.11`, `Acoa.18`); boolean builders are adjectives without
  `is` (`B4.5`, `B12.13`).
- **[MUST]** Builders are nouns (return value); manipulators are verbs returning `void`; never both
  (`B4.3`, `B4.4`, `Adt.13`, `Adt.14`).
- **[MUST]** Every public method implements an interface method (`B1.8`, `B4.8`, `Apa.11`).
- **[MUST]** Keep interfaces short (≤3 methods; flag overloads) (`B4.9`, `Adt.29`, `Acob.22`).
  Convenience goes in a nested `Smart` class (`B4.10`, `B12.15`).
- **[SHOULD]** Depend on the narrowest interface that serves the need (`Iterable` over `Collection`
  over `List`) (`Ada.17`, `Adt.5`). Polymorphism: type collaborators by interface (`Acob.25`).
- **[SHOULD]** Name interfaces as nouns; implementation classes carry a short entity prefix
  (`Rq*`, `Rs*`, `Tk*`, `Default*`, `Mk*`) (`Acob.19`, `Acs.11`).
- **[NICE]** One-word noun names for variables/methods/fields; avoid compounds (`Acs.10`, `Apb.7`).
- **[MUST]** Let names replace documentation; document only external interfaces (`B4.14`, `B8.4`,
  `Apa.31`).
- **[SHOULD]** A returning method is named as the thing it returns, not `find`/`get`
  (`B4.15`, `Adt.14`, `Acoa.20`).
- **[SHOULD]** Avoid giant interfaces and fluent APIs that force class bloat (`Acob.22`).
- **[SHOULD]** Wrap final JDK types in a `Scalar<X>` to expose an object (`Acob.24`).
- **[NICE]** Redesign equality around a comparable byte contract instead of `instanceof`+cast
  (`Acs.14`).
- **[SHOULD]** Use a glossary; one term = one meaning in specs (`Apa.10`).
- **[MUST]** Ban the `*Client` suffix; model the server's entities instead (`Acob.20`).
- **[SHOULD]** Hide parsing inside a parsing object (`implements Book`), not a DTO
  (`Acob.23`).
- **[NICE]** Prefer skinny interfaces returning raw data that adapters dress up (`Ada.16`).
- **[SHOULD]** Prefer printers/media (`print(Media)`) over getters for serialization (`Acoa.20`).
- **[NICE]** Avoid overloading when two behaviors share a name; prefer distinct names or decorators
  (`Acs.12`) — default follows the book (`B4.6`) which endorses overloading that funnels to one
  implementation.

## 5. Decoration & composition

- **[MUST]** Add behavior by wrapping (decorator/envelope), never by implementation inheritance
  (`B5.1`, `B5.4`, `B5.5`, `Ada.18`).
  ```java
  final class Encrypted implements Document {
    private final Document plain;
    Encrypted(Document d) { this.plain = d; }
    public byte[] content() { return encrypt(this.plain.content()); }
  }
  ```
- **[MUST]** Prefer an interface + decoration; `final` classes, `final` delegate methods on envelope
  bases (`B5.3`, `TextEnvelope.java:14-48`).
- **[MUST]** Inherit only to refine an `abstract` class, whose non-extension methods are `final`
  (`B5.2`, `B5.12`, `Acoa.24`).
- **[SHOULD]** A decorator's state mirrors the wrapped object; it may expose the same interface or a
  different one (`B5.6`). `Smart` adds methods; a decorator overrides them (`B5.7`).
- **[MUST]** Replace `if`/`for`/`switch`/`while` with composable objects where practical (`B5.9`,
  `B8.6`); move branch-if-you-run logic into a decorator (`Acs.17`, `Acs.30`).
- **[MUST]** Never use static methods as reusable components (`B5.10`).
- **[SHOULD]** Encapsulate a third-party static utility in a small object (`B5.11`, `B11.5`).
- **[SHOULD]** Replace fluent chains with nested decorators (`Acob.29`):
  ```java
  new BodyOfResponse(new ResponseAssertStatus(new RequestWithMethod(req, "GET"), 200)).toString();
  ```
- **[SHOULD]** Validate with decorators, not inline checks (`Acs.16`, `Acs.21`, `Acs.41`).
- **[NICE]** A chain of single-purpose objects (`PsChain` → `PsCookie` → …) replaces a switch
  (`Apa.17`).
- **[NICE]** Decorate the small retrieved object, not the whole collection (`Apb.10`).
- **[MUST]** Reject traits/mixins; layer decorators instead (`Acob.26`).
- **[SHOULD]** Subtyping via `interface Article extends Manuscript` is legitimate; copying
  implementation is not (`Acob.28`).
- **[SHOULD]** Build responses/objects by stacking small decorators, one concern each
  (`Adt.17`, `Acob.27`).
- **[SHOULD]** Move X-printing/marshalling into a decorator of the object (`Adt.18`).
- **[NICE]** Make a custom `Iterator` a read-only adapter; leave it unsynchronized
  (`Ada.19`, `Ada.20`).
- **[SHOULD]** Use decorators for cross-cutting behavior (retry/log) instead of try/catch loops
  (`Atq.7`).
- **[NICE]** Method chaining that builds new objects (`book.pages().last().text()`) is legal OO; the
  Law of Demeter only bans getter-reaching (`Acs.18`).
- **[SHOULD]** Separate parsing/printing via a decorated `Template`, not DTO + utility formatter
  (`Acs.20`).
- **[MUST]** Avoid the Builder pattern; a growing argument list means extract collaborators, not a
  builder (`B5.8`, `B12.14`).

## 6. Exceptions & null

- **[MUST]** Never accept `null`; use a Null Object or a `Mask`-style object (`B6.1`, `B6.13`).
- **[MUST]** Never return `null`; choose: throw, return an empty collection, or a Null Object
  (`B6.3`, `B12.5`).
- **[MUST]** Do not defend inline against `null` that arrives anyway; let the NPE happen (or use a
  validating decorator) (`B6.2`, `Acs.21`).
- **[MUST]** Fail fast: never return sentinel values (`-1`, `0`) or safe defaults (`B6.4`, `B6.12`,
  `Atq.10`).
- **[MUST]** Throw only checked exceptions; one exception type is enough (`B6.5`, `B6.9`, `Atq.9`).
- **[MUST]** Don't catch unless to chain and rethrow; wrap the cause (`B6.6`, `B6.7`, `B6.16`,
  `Atq.8`).
  ```java
  catch (IOException ex) { throw new Exception("Can't read " + file, ex); }
  ```
- **[MUST]** Recover exactly once, at the top-level entry point (`B6.8`).
- **[MUST]** Never catch-and-log; never use exceptions for flow control (`B6.10`, `B6.15`).
- **[MUST]** Avoid `java.util.Optional` as a null replacement (`B6.14`, `B11.6`).
- **[SHOULD]** Put maximum context in every message, naming the concrete object/identifier
  (`Atq.11`).
- **[SHOULD]** One catch block per exception originator; keep `try` as small as the throwing call
  (`Atq.12`, `Atq.13`).
- **[MUST]** Restore the interrupt flag before rethrowing `InterruptedException` (`Atq.14`).
- **[MUST]** Never use Java `assert` for runtime checks (`Atq.15`).
- **[SHOULD]** Retry transient operations via a bounded policy/decorator, not ad-hoc loops
  (`Atq.16`, `Adt.21`, `B6.11`).
- **[MUST]** Unsupported operations throw `UnsupportedOperationException`; exhausted iterators throw
  `NoSuchElementException` (`Ada.22`, `Ada.23`).
- **[MUST]** Never write code after a `throw`; no dead `else` (`Acs.22`).
- **[SHOULD]** Inject the logger; default to a no-op `Log.NULL` (`Acob.33`).
- **[SHOULD]** A Null Object answers common calls benignly and throws only on entity-specific ones
  (`Acoa.26`, `Acoa.27`, `B6.13`).
- **[SHOULD]** Replace `Map.get()`-returns-null with a finder/Iterator (`Acoa.28`).
- **[SHOULD]** Prefer a default-argument overload over NULL/exception-on-empty (`Acob.31`).
- **[MUST]** Fail non-zero exit codes with a `Safe` decorator (`Adt.20`).
- **[SHOULD]** Wrap checked IO in `IoCheckedScalar`; keep ctors code-free (`Ada.24`).
- **[MUST]** Let invalid input crash; put protection in decorators (`Acs.21`).
- **[SHOULD]** When a bug cannot be reproduced, add a passing test proving intended behavior and
  close the ticket (`Apa.20`, `Apa.21`, `Apa.27`).
- **[MUST]** Kill `null` as a refactoring step; only keep JDK-forced cases (`Apb.12`).

## 7. Testing

- **[MUST]** A unit test is part of the class; classes without tests don't ship (`B7.1`, `B9.2`).
- **[MUST]** One statement per test: construct the subject, then a single `assertThat(reason, actual,
  matcher)`; arrange by composition, not imperative setup (`B7.3`, `Atq.21`, `Atq.22`).
  ```java
  assertThat("total", new Phrases("Hi!").greetings().count(), equalTo(1));
  ```
- **[MUST]** No `@Before`/`@BeforeClass`, no shared fixtures/fields; give each test its own literals
  (`Atq.21`, `Atq.22`, `ProhibitFieldsInTestClassesCheck`).
- **[MUST]** Prefer fakes over mocks; ship a nested `Fake` (or `Mk*`/`*Mocker`) implementation with
  each interface, in `src/main` when it is a legitimate object (`B7.4`, `B7.5`, `Atq.23`, `B12.7`).
- **[MUST]** Tests assert public behavior only, never internal interactions (`B7.7`).
- **[MUST]** When an interface changes, its fake changes; the test does not (`B7.8`).
- **[MUST]** Treat test code with the same care/limits as production (`B7.9`, `B8.10`).
- **[MUST]** Every bug fix ships with a reproducing test; report failures as `@Disabled` tests + a
  `@todo` puzzle (`B7.12`, `Atq.34`, `Apa.27`).
- **[MUST]** Assert with Hamcrest/`MatcherAssert.assertThat`, not `Assert.*`; use OO matchers for
  XML/HTTP (`Adt.22`, `Atq.42`).
- **[SHOULD]** Keep test classes `final`, package-private, named after the class they validate
  (`FooTest`/`FooITCase`); methods are behavior sentences, no `test` prefix (`Atq.5`, `Atq.6`,
  `JUnitTestClassShouldBeFinal`).
- **[SHOULD]** Externalize bulky expectations to resource files; parametrize genuinely shared
  matrices (`Acob.35`, `Atq.30`).
  ```java
  // src/test/resources/.../sample.xml -> <sample><spec/><xpaths/></sample>
  ```
- **[SHOULD]** Run slow/deep tests only in CI, keep the local loop fast; tag or profile them
  (`Atq.30`, `Atq.54`, `Atq.80`).
- **[SHOULD]** Test HTTP against a real server/socket on a random port or a mock container
  (`Atq.38`, `Adt.53`).
- **[SHOULD]** Guard optional tooling/resources with `Assumptions`, never delete the test
  (`CompilerTest.java:154-168`).
- **[SHOULD]** Use property-based tests for parsers/serializers; run cross-implementation ITs where a
  pluggable provider exists (`XemblerTest.java:165`, xembly `src/it/{saxon,xerces}`).
- **[NICE]** Prove concurrency with a latch-driven parallel test (`Atq.31`, `Threads<>`).
- **[NICE]** More test code than production code is healthy (1:2 is fine) (`Atq.39`).
- **[NICE]** Do not refactor while fixing a failing test; file the refactor separately (`Atq.40`,
  `Atq.77`).
- **[SHOULD]** Build test scaffolding with fake objects/extensions, not `private static` helpers
  (`Atq.23`, `Atq.36`).
- **[MUST]** Measure coverage on every build and fail below the threshold (`Apa.26`).
- **[NICE]** A missing test is a bug; a flaky test is a bug (`Atq.19`, `Atq.20`).
- **[SHOULD]** Test only what you actually care about; no `notNullValue()` after `toString()`
  (`Atq.42`).
- **[NICE]** Keep test literals distinct; don't hoist a shared constant (`Atq.43`).
- **[SHOULD]** Start each task from a failing test that reproduces the problem (`Apa.23`,
  `Apb.13`).
- **[NICE]** Submit tests separately from the fix to prevent "fix the tests to pass" (`Atq.33`,
  `Atq.58`).
- **[NICE]** Test your test doubles (`Atq.26`).
- **[MUST]** No logging inside a unit test; assert instead (`Atq.27`).
- **[SHOULD]** Split unit (`*Test`, surefire) from integration (`*ITCase`, failsafe); run ITs against
  a real dependency (`Atq.56`, `Adt.23`, `Adt.40`).

## 8. Code style

- **[MUST]** LF line endings, no tabs, no trailing spaces, no two consecutive blank lines, no blank
  line before a closing brace, file ends in a newline (Qulice `checks.xml`).
- **[MUST]** Line length ≤100 (some repos ≤80); no line ends with an operator; no line starts with
  `=`.
- **[MUST]** Paired brackets; monotonic indentation; fluent multi-line calls stay attached to the
  previous line (`Adt.31`, `Acs.25`, `Acs.26`).
- **[MUST]** No star imports, no static imports, no redundant/illegal imports; ordered, cohesive
  imports.
- **[MUST]** No `++`/`--` expressions; no inline conditionals; no `clone`/`finalize`; `RequireThis`
  on fields.
- **[MUST]** One `return` per method (`ReturnCount=1`, `Apb.8`).
- **[MUST]** ≤3 parameters; ≤40 executable statements; lambda body ≤20; class fan-out limited.
- **[MUST]** No blank lines inside method bodies; an empty line means "extract a method/class"
  (`Acs.24`, `Acs.39`).
- **[MUST]** Full Javadoc on every type/method/field with ordered, capitalised tags (`@param`,
  `@return`, `@throws`, `@since`) — Qulice requires it (`knowledge/elegant-objects-java.md`; Takes has 424 `@since`).
- **[MUST]** No comments narrating internals; names and structure carry meaning (`B8.4`, `B8.5`).
  The tension with `Acs.27` ("no Javadoc at all") is resolved in favour of documenting external
  interfaces while banning inline narration.
- **[MUST]** `final` classes, `private final` fields, `final` parameters and locals; `this.` prefix.
- **[SHOULD]** No single-use private constants (inline them); no one-time variables (`Adt.30`,
  `Acs.8`).
- **[SHOULD]** Simple beats clever; assume a junior reader (`B8.3`); optimize for a stranger's speed
  (`Acs.23`, `Acs.28`).
- **[SHOULD]** No string `+` concatenation for messages; use `String.format` with indexes (`Adt.71`).
- **[MUST]** No shared public constants/enums; use micro-classes (`B1.15`, `B4.12`, `B8.7`,
  `Acoa.33`, `B12.1`).
- **[SHOULD]** Ban auto-formatting that hides the rules; contributors fix style manually (`Acob.38`).
- **[SHOULD]** Keep one language/technology per repository for one enforced style (`Ada.29`).
- **[SHOULD]** Decompose before measuring; compute LoC/cohesion/coupling per small repo (`Ada.30`).
- **[NICE]** Use concrete cohesion/coupling metrics (jPeek) to validate EO claims (`Acob.40`,
  `Acob.41`).
- **[SHOULD]** Fewer language options track higher quality; forbid syntactic sugar (`Atq.48`).

## 9. Static analysis & quality gates

- **[MUST]** Run Qulice in the build (goal `check`, `verify` phase); any violation fails the build
  (`B8.9`, `Atq.44`, `CheckMojo.java:187-191`).
- **[MUST]** Static analysis is opt-in by profile locally (`-Pqulice`) but mandatory in CI and
  release (`mvn --errors --batch-mode clean install -Pqulice`).
- **[MUST]** Bundle and lock the rule set; allow only exclusions and per-project `-Xep` overrides,
  never rule redefinition (`knowledge/elegant-objects-java.md` §C).
- **[MUST]** Emit actionable violations (`validator: file[line]: message (RuleName)`).
- **[MUST]** Enforce exactly-one-assert tests, `final` test classes, no plain JUnit asserts
  (`ruleset.xml:578-639,824-849`).
- **[MUST]** Gate coverage and mutation in the POM (Jacoco + PIT; ~0.65/75 is realistic) (`Atq.50`).
- **[SHOULD]** Gate API compatibility with Revapi; each accepted break carries a written
  `<justification>` (`knowledge/elegant-objects-java.md`; Takes `pom.xml:353-687`).
- **[SHOULD]** Ratchet strict rules: ban a static API only for already-migrated packages, grow the
  list via a PDD puzzle (`forbiddenapis` + `@todo #1434`).
- **[SHOULD]** Lint test code too (`jtcop`) and enforce architecture (`ArchUnit`) on large projects.
- **[MUST]** When two analyzers contradict, disable exactly one rule with a written rationale; never
  blanket-`@SuppressWarnings` (`knowledge/elegant-objects-java.md` §H).
- **[MUST]** Never leave a suppression that covers nothing; unused suppressions are violations.
- **[SHOULD]** Prefer multiple small quality workflows/checks over one mega-report.
- **[MUST]** Make the quality wall impossible to go around — automated pre-flight + read-only master
  (`Atq.52`).
- **[SHOULD]** Finalize public methods so subclasses cannot break contracts (`Atq.45`).
- **[SHOULD]** Treat inconsistent style and missing docs as bugs (`Atq.46`).
- **[SHOULD]** Prioritize maintainability over functionality under constraints (`Atq.47`).
- **[MUST]** Make the CI pipeline as fragile and strict as possible (`Apa.29`).
- **[MUST]** Eliminate the seven maintainability sins: anti-patterns, untraceable changes, ad-hoc
  releases, volunteer static analysis, unknown coverage, nonstop development, undocumented interfaces
  (`Apa.30`, `Apa.122`).
- **[SHOULD]** Keep repositories small so lint/tests/review can be maximal (`Atq.51`).
- **[NICE]** Carrot-and-stick for AI agents: a manifesto plus hard checkers (`Apb.19`).

## 10. Build & Maven

- **[MUST]** Inherit a shared parent POM (`com.jcabi:parent`) for cross-cutting policy; keep the repo
  POM to domain deps + profiles (`Adt.74`).
- **[MUST]** One non-interactive build command used by CI and documented in the README:
  `mvn --errors --batch-mode clean install -Pqulice`.
- **[MUST]** Pin `maven.compiler.release` and make it identical to the CI JDK and the documented JDK
  (the s3auth 1.8/21/11 drift is the counter-example).
- **[MUST]** Keep the version `-SNAPSHOT` in source; the release bot injects the tag via
  `mvn versions:set -DnewVersion=${tag}` (`Adt.36`).
- **[SHOULD]** Centralize versions in `dependencyManagement`/a BOM; children omit versions
  (`knowledge/elegant-objects-java.md` §C).
- **[SHOULD]** Use named profiles by purpose (`qulice`, `jacoco`, `deep`, `sonar`, `sonatype`,
  `site`), never by environment.
- **[MUST]** A library has zero non-`provided` runtime dependencies where possible; exclude
  transitive duplicates (including self) (`pom.xml:74-79`, xembly `pom.xml:69-80`).
- **[MUST]** Compose `argLine` from an empty property (`<argLine/>` + `@{argLine} …`) so coverage
  agents can inject (`knowledge/elegant-objects-java.md` §H).
- **[SHOULD]** Verify repository invariants as build steps (license header at `verify`, API compat,
  mutation) (`knowledge/elegant-objects-java.md` §C).
- **[SHOULD]** Provision integration infrastructure via profiles/plugins (DynamoDBLocal, reserved
  ports), not mocks (`Adt.41`, `Adt.39`).
- **[SHOULD]** Wire generated resources (SCSS→CSS, XSL) into the build; pass build identity through
  the JAR manifest (`Manifests.read`), not constants (`Acs.32`, `Acs.36`).
- **[NICE]** Commit `.mvn/jvm.config` / a Maven wrapper for reproducible builds (newer repos only).
- **[SHOULD]** Split builds by purpose/cost: fast local, cheap CI, preflight merge, proper release
  (`Adt.42`).
- **[SHOULD]** Run every build in a disposable container (`Adt.33`); run as non-root inside it
  (`Adt.34`).
- **[SHOULD]** Write your own `release.sh` (or bot command) so the whole release is one command
  (`Apa.34`, `Apa.35`).
- **[MUST]** Tag/publish every release; every version stays downloadable (`Apa.36`).
- **[SHOULD]** Prefer several small releases a day over rare big ones (`Apa.37`, `Ada.32`).
- **[NICE]** Pin dependency versions by trust (dynamic ranges only for trusted authors) (`Ada.31`,
  `Ada.44`).
- **[SHOULD]** Separate unit and integration phases; fail the build at `verify` (`Atq.56`,
  `Adt.40`).
- **[NICE]** Version DB schema with changesets; keep credentials in Maven profiles, not POMs
  (`Adt.47`, `Adt.48`).
- **[NICE]** Publish a Maven-Central JAR that includes the fakes/tests where they are part of the
  contract (`Acoa.38`).
- **[NICE]** Keep build definitions deduplicated; a parent POM is the fix for copy-pasted plugin
  config (`Adt.74`).
- **[SHOULD]** Make the build reproducible: pin versions, UTF-8, committed JVM config, wrapper
  (`knowledge/elegant-objects-java.md` §C).

## 11. CI/CD & release

- **[MUST]** Split pipelines: GitHub Actions is the PR gate (lint + build + tests + coverage); **only**
  the bot (Rultor) merges and releases (`knowledge/elegant-objects-java.md` §D.1; `Apa.32`).
- **[MUST]** `master` is read-only; merges are PR-only and bot-validated (`Apa.32`, `Apa.50`).
- **[MUST]** One small, single-purpose workflow per concern — build, coverage, static analysis,
  duplication, license, typos, YAML/Markdown/XML lint, dependency audit, docs-version (`Atq.53`).
- **[MUST]** Every workflow declares `permissions: contents: read` by default, a `concurrency`
  cancel-in-progress group, `timeout-minutes`, and a Maven cache keyed on `hashFiles('**/pom.xml')`.
- **[SHOULD]** Use restore-only caches on PR workflows; `rm -rf ~/.m2/.../org/<group>` before install
  for multi-module snapshots.
- **[SHOULD]** Test across OS × one modern JDK (ubuntu/windows/macos) for portability; declare
  capability skips instead of letting one OS dictate.
- **[MUST]** Release is tag-driven and scripted: validate `^[0-9]+\.[0-9]+\.[0-9]+$`, `versions:set`,
  commit the tag, signed `deploy -Psonatype`; never ad-hoc (`Apa.34`, `Apa.36`).
- **[MUST]** Secrets never touch history: assets from a separate repo, `sensitive:` markers,
  copy→commit→push→reset, `trap EXIT` in manual scripts (`Adt.38`).
- **[SHOULD]** Deploy app repos by `git push -f` to the PaaS remote, then curl the live URL with
  retries; release is not done until the endpoint answers (`Adt.49`).
- **[SHOULD]** Keep a `Procfile`/Dokku config/runtime declaration in-repo, plus a manual `deploy.sh`
  fallback.
- **[MUST]** Don't merge into a broken master; a build fix is its own PR (`Apb.21`, `Apb.22`,
  `Apb.73`).
- **[SHOULD]** Automate doc/version sync with an `up.yml` PR from the latest tag.
- **[SHOULD]** Keep dependencies moving with Renovate (guard-rails for pins that must not float).
- **[NICE]** Prefer several small releases a day; every version stays downloadable (`Apa.36`,
  `Apa.37`).
- **[SHOULD]** Add CI/coverage badges immediately and keep them green (`Apb.26`).
- **[SHOULD]** A green branch build is not enough; only a merge gate on the write path protects
  master (`Apa.33`).
- **[NICE]** Verify Windows builds on a second CI (AppVeyor/AppVeyor-style) before merge
  (`Adt.43`, `Adt.52`).
- **[NICE]** Simulate production and run stress tests at the highest maturity levels (`Adt.54`).
- **[SHOULD]** Release components independently, each with its own repo/build/README/license
  (`Ada.33`).

## 12. Repository & docs conventions

- **[MUST]** SPDX header on every source and config file; enforce with `license-maven-plugin` + REUSE
  + a `reuse` workflow; `REUSE.toml` maps non-source globs.
- **[MUST]** One license stated identically in POM, SPDX, REUSE, README badge and `LICENSE.txt`
  (requs's BSD-vs-MIT is the counter-example).
- **[MUST]** `.gitattributes` = `* text=auto eol=lf` (+ `*.java ident`, `*.xml ident`); small
  tool-focused `.gitignore` (include `.claude/`, IDE, `target/`, `node_modules/`).
- **[MUST]** A `package-info.java` with a one-line Javadoc in every package.
- **[MUST]** Elegant README as the external contract: problem statement in the first paragraph,
  badges (EO principles, Rultor, CI, coverage, license), install snippet, usage cookbook, Architecture
  section, "how to contribute" with the exact `mvn … -Pqulice` command (`Apb.35`, `Apb.36`).
- **[SHOULD]** README lines ≤80, second-level headers only, one blank line between blocks, no
  license/changelog/contributor boilerplate in the opening (`Apb.37`).
- **[MUST]** Everything traceable: every change has a GitHub issue; commits start with `#123` so the
  tracker auto-links (`Apb.30`, `Apa.107`).
- **[MUST]** Never rewrite history: no force-push, no deleted commits/comments (`Apa.9`).
- **[SHOULD]** Use PDD: `@todo #N:30min …` puzzles + 0pdd issues; no bare `TODO`/`FIXME`
  (`Apa.39`–`Apa.42`).
- **[NICE]** One change = one small PR (<50 hits-of-code for newcomers) (`Apb.38`).
- **[SHOULD]** Address comments with `@nickname`; let the reporter close the ticket (`Apb.39`,
  `Apb.41`).
- **[NICE]** Keep docs as versioned Markdown reviewed via PRs; `CITATION.cff` for research output
  (`Apa.106`).
- **[SHOULD]** Record key decisions with rejected alternatives; list assumptions/risks/concerns
  (`Apa.73`, `Apa.74`).
- **[SHOULD]** Document external interfaces, not internals; working software beats internal docs
  (`Apa.31`).
- **[NICE]** Make master's build status visible via badges (the pipeline is watched) (`Apb.26`).
- **[SHOULD]** Never leave the build red; report a broken master and wait, or fix it in its own PR
  (`Apb.21`, `Apb.73`).

## 13. Project & process

- **[MUST]** Every ticket is a complaint with a minimal reproduction; no "BTW" scope creep
  (`Atq.59`, `Atq.61`, `Atq.62`).
- **[MUST]** Communicate through tickets, not meetings/chat/email; the task author is the single
  point of contact (`Apa.51`, `Apa.52`).
- **[SHOULD]** Decompose into 30–60-minute micro-tasks with a fixed micro-budget; overflow becomes
  puzzles (`Apb.31`, `Apb.32`).
- **[SHOULD]** Define Done as the author's acceptance; pay only for closed deliverables, never for
  hours (`Apa.45`, `Apa.95`, `Apb.50`).
- **[SHOULD]** Run in four phases — Thinking (spec) → Building (one architect's prototype) → Fixing
  (mass bug work) → Using (bug fixes only) (`Apa.54`).
- **[SHOULD]** Start every task with a failing test; if out of time, commit the `@Disabled` test +
  `@todo` and close (`Apa.23`, `Apa.24`).
- **[MUST]** Fix and break at the same time: a deliverable = fix + tests + reported bug/puzzle
  (`Apa.46`, `Apa.48`).
- **[SHOULD]** Report artifacts (not effort); keep a plan with owner + reviewer per artifact
  (`Apb.43`, `Apb.44`).
- **[SHOULD]** Write key decisions with their rejected alternatives; list assumptions/risks/concerns
  (`Apa.73`, `Apa.74`).
- **[NICE]** Estimate cost as price-per-unit (HoC/quality), never a fixed total (`Apa.70`,
  `Apa.119`).
- **[NICE]** Assign personal responsibility; rules derive performance decisions, not mood
  (`Apa.78`, `Apa.76`, `Apa.127`).
- **[MUST]** Every idea starts with a ticket; no changes without code review (`Apb.30`).
- **[SHOULD]** Make bugs welcome and pay for discovering them; one bug per 2–3 completed tasks is
  healthy (`Atq.63`, `Apa.48`).
- **[SHOULD]** Require bug reports or PRs from contributors; nothing else counts (`Atq.64`).
- **[SHOULD]** Let programmers chase speed; the pipeline enforces quality (`Atq.65`).
- **[SHOULD]** Every change ties to an issue and a same-named branch (`Adt.55`).
- **[SHOULD]** Let an automated merge bot merge and report in the PR (`Adt.57`).
- **[SHOULD]** Decompose work top-down by user value, not technical layers (`Ada.35`).
- **[SHOULD]** Write a short Product Vision in four sections; cap features per actor and quality
  requirements (`Apa.55`, `Apa.98`).
- **[MUST]** Keep functional and non-functional requirements separate and measurable (`Apa.13`,
  `Apa.14`).
- **[SHOULD]** Appoint exactly one accountable architect; decisions are documented artifacts
  (`Apa.62`, `Apb.76`).
- **[NICE]** Run independent reviews; pay per bug found; ask for criticism, rotate reviewers
  (`Apa.66`–`Apa.68`).
- **[NICE]** Track Hits-of-Code, not LoC (`Apa.71`, `Acoa.36`).
- **[MUST]** Blame the project for unclear code, not yourself; file docs/source bug tickets
  (`Apb.74`, `Apb.75`).
- **[NICE]** Prefer rules and planning over an unpredictable manager (`Apa.127`).
- **[NICE]** Resolve review conflicts exactly three ways: accept, hold firm, or appeal to the
  architect — never compromise (`Apa.115`).
- **[SHOULD]** A reviewer must prove the code is bad; the author need not prove it good (`Apa.116`).
- **[MUST]** Trust without control becomes chaos: pair delegation with tests/gates/rules
  (`Apa.120`).
- **[NICE]** Avoid the seven project sins that make software unmaintainable (`Apa.122`).

## 14. Tooling (pick-by-need)

- **[MUST]** Java + Maven on `com.jcabi:parent`; JUnit 5 + Hamcrest; Qulice; Jacoco (+ PIT for
  libraries); Revapi for public APIs; Rultor; Renovate; REUSE/SPDX; PDD/0pdd.
- **[SHOULD]** Cactoos for OO primitives; Takes for web apps; jcabi-* for HTTP/XML/JDBC/Dynamo/S3/
  SSH/manifests/logging; Xembly for XML generation; Saxon for XSLT 2.0.
- **[SHOULD]** Fakes (`Fake*`/`Mk*`/`Fk*`) instead of Mockito; `jqwik` for properties; JMH for
  benchmarks; `DynamoDBLocal`/H2 + reserved ports for persistence ITs; `MkContainer`/`FtRemote` for
  HTTP.
- **[NICE]** ArchUnit, jtcop, Sonar, Codacy, Infer, scancode, simian, `hoc`, `tdx` as extra checks
  once the build and coverage gates are in place.
- **[NICE]** Lombok `@EqualsAndHashCode`/`@ToString` on immutable value objects (`provided`); keep
  Guava/Commons behind a wrapper class; never a DI container or ORM.
- **[NICE]** AI agents for mechanical chores (bug rewording, small refactor PRs, docs sync) — one
  small concern per PR (`Apb.70`).
- **[MUST]** Wrap third-party static utilities in objects; keep vendor types behind `Dy*`/`Tk*`
  classes (`B11.5`, `Adt.60`, `Adt.78`).
- **[SHOULD]** Use `@RetryOnFailure`/explicit retry decorators only on idempotent transient
  operations (`Atq.71`, `Adt.21`).
- **[SHOULD]** Use `XhtmlMatchers` for XML assertions (`Atq.68`).
- **[SHOULD]** Use JUnit 5 tags/extensions/`@Disabled` rather than home-grown categorisation
  (`Atq.69`).
- **[MUST]** Automate PDD with the `pdd` gem + 0pdd, not a manager (`Apa.99`).
- **[NICE]** Use Requs for machine-checkable SRS and `hoc` for effort metrics (`Apa.100`,
  `Apa.101`).

---

## Appendix A — Anti-pattern catalogue (do not copy)

These appear in the analysed legacy repos; treat them as migration targets, never as templates.

| Anti-pattern | Evidence | Replace with |
|---|---|---|
| Static utility parser / God class | xembly `Verbs.java:30-56` | small collaborating objects |
| Mutable fluent builder | xembly `Directives` ("mutable and thread-safe") | immutable `with(...)` or constructor args |
| Mock returning `null` | xembly `MkHost.stats()` | Null Object |
| Utility class with private ctor | requs `Entrance.java:26-33`, `Main.java:22-29` | a thin launcher delegating to objects |
| Static methods/fields | requs `Compiler.decor`, `Compiler.SCHEMA`, `TkApp.REV` | constructor injection / `private static final` constant only |
| Public mutable plugin fields | requs `CompileMojo.java:41` | confined to the Maven Mojo API |
| Internal `null` + `assert` | requs `Retry.call`, `Compiler.java:99` | Null Object / throw |
| JDK/CI/docs drift | s3auth `pom.xml:268-269` vs CI 21 vs README 11 | one pinned `release`, aligned |
| License metadata drift | requs BSD POM vs MIT SPDX | one license everywhere |
| Hard-coded machine path in POM | xembly `prof` `pom.xml:472` | local profile only |
| Blanket `@SuppressWarnings` without reason | avoid entirely | narrow, reasoned, ticket-linked exclusion |

## Appendix B — Source disagreement resolution

| Topic | Sources disagree | Default rule | Exception |
|---|---|---|---|
| Method overloading | Book `B4.6` vs `Acs.12` | overload genuine convenience funnelling to one implementation | ban overloads when semantics differ |
| Law of Demeter / chaining | `Acs.18` vs `Acob.22,29` | nested decorators for APIs you own | chaining objects that build new objects is fine |
| Prefixes vs compound names | `Acs.11`/`Acob.19` vs `Acs.10`/`B4.2` | short entity prefixes (`Rq*`, `Tk*`) | plain one-word nouns to prefer when possible |
| "Let it crash" vs no-null | `Acs.21`/`Atq.10` vs `B6.2` | fail fast on non-null invalid state; never tolerate null | JDK/third-party null guards allowed in a decorator |
| AOP/annotations | `B6.11`/`B11.4` vs `Adt.2`/`Adt.21` | visible decorator | aspect only when weaving is unavoidable |
| Test-first vs bug-driven | `Ada.25` vs `Atq.41` | bug-driven: reproduce, then fix | test-first where the requirement is clear |
| Test-class size | `B1.21`/`B8.10` vs `Atq.37` | focused per behavior | a long test class mapping 1:1 is fine |
| Documentation | `B8.4` vs `Acs.27` | document external interfaces; no inline narration | machine-checkable interpretability gate as an option |
| Dependency versions | `Ada.31` vs `Apb.23` | pin exact + Renovate guard-rails | dynamic ranges only for trusted authors |
| Managed repo vs S3 | `Adt.46` vs `Apb.67` | managed cloud repo | S3 wagon for legacy/internal |

---

### How to use this rulebook

1. **Encode the [MUST] rules as static checks** (Qulice plugin/custom rules + CI gates) so they fail
   the build — a rule that isn't enforced will be ignored (`Apa.109`).
2. **State the [SHOULD] rules in the agent's manifesto** (`SKILL.md`/`AGENTS.md`) and make them the
   default in generated code; deviations require a written reason (an `@checkstyle`/`@SuppressWarnings`
   with a ticket).
3. **Treat the [NICE] rules as review guidance** — prefer them, but never block a correct change on
   them.
4. **When sources disagree, follow the stated default and cite the exception** — the full arguments
   (with `path:line` evidence) live in `knowledge/elegant-objects-java.md`.
