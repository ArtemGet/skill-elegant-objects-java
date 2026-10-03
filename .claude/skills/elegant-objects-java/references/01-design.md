# 01 — Design (Elegant Objects, Java 17)

> **When to use this reference**
> - Designing any new class, interface, or object graph in this codebase.
> - Refactoring legacy Java (DTOs, `-er` managers, statics, builders, getters) toward EO.
> - Reviewing a PR for EO violations before it reaches Qulice/CI.
> - Deciding where behavior belongs: constructor vs. method vs. decorator.
> - Writing or fixing exception/null handling at an API boundary.

Priority tags: **[MUST]** build-breaking · **[SHOULD]** strong default, deviate only with a written
reason · **[NICE]** improve elegance, judge case by case. Rule ids cite `knowledge/elegant-objects-java.md`.

---

## 1. Core OOP & design

A class is not a bag of functions; it is a factory of **representative objects** — living stand-ins
for real-life entities (`B1.1`, `B1.2`, `Acoa.1`). If you cannot name or draw the entity an object
stands for, stop and re-model. Prefer many small noun-named objects over a few large procedural ones.

### Rules

- **[MUST]** Model a real entity with coordinates; a stateless class is isomorphic to a static
  method and is not an object (`B1.7`, `Acoa.5`).
- **[MUST]** Encapsulate state; state is identity. Derive `equals`/`hashCode` from those coordinates
  (`B1.5`, `B3.13`).
- **[MUST]** Keep data and its processor in the same object; no request/response DTOs, no anemic
  records passed to a "service" (`Ada.4`, `Acoa.49`).
- **[MUST]** Compose smaller objects into bigger ones; prefer declarative assembly over imperative
  step-by-step code (`B1.17`, `B8.6`).
- **[MUST]** Reject anti-patterns early: God object, Singleton, utility class, global mutable state,
  MVC controller, Builder, Facade, Visitor, Template Method (`Apa.4`, `Acoa.25`, `B12.23`).
- **[SHOULD]** Represent the smallest real entity: a no-arg ctor that "can do everything" stands for
  "the Universe" (`Acoa.6`, `Acoa.9`).
- **[SHOULD]** Decompose vertically (decorators) instead of horizontally (client-assembled siblings)
  (`Acob.1`).

### Smells → fix

| Smell | Fix |
|---|---|
| `XxxUtils` / `XxxManager` with static methods | Real object with injected collaborators (`B1.9`, `B1.11`) |
| `class UserData { String name; int age; }` | `final class User implements Person` with behavior (`Ada.4`) |
| Static `main`-runner doing all logic | Thin launcher delegating to composed objects (`B12.23`) |
| `instanceof` / cast chains in business logic | An interface method per variant (`B1.14`) |

### BEFORE / AFTER

```java
// BEFORE — anemic data bag + static utility (not OO at all)
final class UserData {
    String name;
    int age;
}
final class UserUtils {
    private UserUtils() { }
    static boolean adult(UserData data) {
        return data.age >= 18;
    }
}
```

```java
// AFTER — the entity carries its own behavior
interface Person {
    boolean adult();
}
final class User implements Person {
    private final String name;
    private final int age;
    User(String label, int years) {
        this.name = label;
        this.age = years;
    }
    @Override public boolean adult() {
        return this.age >= 18;
    }
}
```

---

## 2. Constructors & lifecycle

Constructors are the software; statements are not (`B2.12`). A class is assembled through
constructors, and each one is **code-free**: it only assigns fields or delegates (`B2.1`). Work is
deferred to methods so objects stay lazy and controllable.

### Rules

- **[MUST]** Constructors only assign fields or delegate via `this(...)`/`super(...)`. No parsing,
  conversion, validation, I/O, regex, `String.format`, or `new` of collaborators (`B2.1`, `B2.5`).
- **[MUST]** Exactly one **primary constructor**, declared last; every secondary funnels to it via
  `this(...)` (`B2.2`, `B2.3`, `B2.4`).
- **[MUST]** `new` is allowed **only in secondary constructors**; methods and the primary receive
  already-built collaborators (`B1.19`, `B2.9`).
- **[MUST]** Instantiate, do not validate, in the primary ctor; validation is a behavior method
  (`B2.14`).
- **[MUST]** An immutable object is complete after construction — no skeleton + setters, no
  `init()`/`setup()`/`open()` step (`B2.15`, `Acs.4`, `Acs.40`).
- **[MUST]** Never call an overridable method from a constructor (fragile base class) (`Acs.6`).
- **[SHOULD]** Defer expensive work (parse, read, format) to the method that needs it; add a caching
  decorator when repeat cost matters (`B2.7`, `B2.8`).
- **[SHOULD]** Inject dependencies only through constructors — never field/setter injection
  (`Ada.8`, `Ada.10`).

### Smells → fix

| Smell | Fix |
|---|---|
| `this.x = Integer.parseInt(s)` in ctor | Wrap `s`; parse in the behavior method (`B2.1`) |
| Duplicated assignment across three ctors | Secondary ctors call `this(...)` (`B2.3`) |
| `new Foo()` inside a method | Inject a factory/envelope; `new` only in secondary ctors (`B2.9`) |
| `init()` called after `new` | Complete the object in one ctor (`B2.15`) |

### BEFORE / AFTER

```java
// BEFORE — parsing in the ctor; initialization duplicated in two places
final class Cash {
    private final int dollars;
    Cash(String text) {
        this.dollars = Integer.parseInt(text.replace("$", ""));
    }
    Cash(float value) {
        this.dollars = (int) value;
    }
}
```

```java
// AFTER — code-free ctors; secondary delegates to one primary declared last
interface Money {
    int dollars();
}
final class Cash implements Money {
    private final String source;
    Cash(float value) { this((int) value); }        // secondary
    Cash(String text) { this.source = text; }       // primary, declared last
    @Override public int dollars() {
        return Integer.parseInt(this.source.replace("$", ""));
    }
}
```

---

## 3. Immutability & state

Every field is `private final`; the object never changes after construction — mutations return a new
object (`B3.1`, `B3.2`). Immutability is not constancy: an immutable object may answer differently
over time because it holds only *coordinates* (a URL, a path), not the data. This is **loyalty**
(`B3.11`, `Acob.14`).

### Rules

- **[MUST]** All fields `private final`; no setters; a mutating operation returns a new object
  (`B3.1`, `B3.2`).
- **[MUST]** No unset `null` fields; split into small cohesive classes instead (`B3.7`).
- **[MUST]** Model mutable resources (disk, network, memory) as immutable objects holding their
  coordinates (`B3.14`, `B3.15`).
- **[MUST]** Grow by replacement: `with(...)` chains or a `Temporary.back()`, never in-place
  mutation (`Acob.17`).
- **[MUST]** Never expose mutable internal data from a method; return unmodifiable/immutable
  collections at boundaries (`ArrayIsStoredDirectly`).
- **[SHOULD]** Separate *state* (stable identity) from *behavior* (frequently changing data) so
  changing data does not force mutation (`Acoa.10`).
- **[SHOULD]** Make thread safety a decorator (`Sync*`); never trust a concurrent field to make a
  compound operation atomic (`Acob.16`).

### Smells → fix

| Smell | Fix |
|---|---|
| Non-final field + setter | `private final` + method returning new instance (`B3.1`) |
| Method mutates `this`, returns `void` | Return a new object (`Acob.17`) |
| Field starts `null`, filled later | Constructor-inject it; split the class (`B3.7`) |
| Returns internal `List` directly | Return `List.copyOf(...)` (`ArrayIsStoredDirectly`) |

### BEFORE / AFTER

```java
// BEFORE — mutable state, temporal coupling, not thread-safe
final class Cash {
    private int dollars;
    void setDollars(int sum) {
        this.dollars = sum;
    }
}
```

```java
// AFTER — immutable; "mutation" produces a new object
final class Cash {
    private final int dollars;
    Cash(int sum) {
        this.dollars = sum;
    }
    Cash mul(int factor) {
        return new Cash(this.dollars * factor);
    }
}
```

---

## 4. Names, interfaces & contracts

A class name says what the object **is**, not what job it performs (`B4.1`). Every public method
implements an interface method (`B1.8`, `B4.8`), and interfaces stay short (`B4.9`). Names replace
documentation; document only external interfaces (`B4.14`).

### Rules

- **[MUST]** Class name = the entity; no `-er`/`-or` job titles. `FileReader` → `DataFile`,
  `PrimeFinder` → `PrimeNumbers` (`B4.2`, `Acoa.16`).
- **[MUST]** No `get`/`set` prefixes; **tell, don't ask** (`B4.11`, `Acoa.18`).
- **[MUST]** Every public method implements an interface method; keep interfaces ≤3 methods (`B1.8`,
  `B4.9`).
- **[MUST]** Ban the `*Client` suffix; model the server's entities instead (`Acob.20`).
- **[MUST]** Builders are nouns (return a value); manipulators are verbs returning `void`; never
  both in one method (`B4.3`, `B4.4`).
- **[MUST]** Let names replace documentation; document only external interfaces (`B4.14`, `B8.4`).
- **[SHOULD]** Name interfaces as nouns; implementation classes carry a short entity prefix (`Rq*`,
  `Rs*`, `Tk*`, `Default*`) (`Acob.19`).
- **[SHOULD]** A returning method is named as the thing it returns, not `find`/`get` (`B4.15`,
  `Acoa.20`).

### Smells → fix

| Smell | Fix |
|---|---|
| `class UserManager` / `class FileReader` | `class Users` / `class TextFile` (`B4.2`) |
| `String getName()` | `String name()` (`B4.11`) |
| 12-method `Service` interface | Split into ≥4 narrow interfaces (`B4.9`) |
| `boolean isAdult()` | `boolean adult()` (adjective, no `is`) (`B4.5`) |

### BEFORE / AFTER

```java
// BEFORE — job-title type, getter, no contract
final class FileReader {
    String read(String path) throws IOException {
        return Files.readString(Path.of(path));
    }
}
```

```java
// AFTER — entity name, noun interface, tell-don't-ask
interface Content {
    String text() throws IOException;
}
final class TextFile implements Content {
    private final Path path;
    TextFile(Path path) {
        this.path = path;
    }
    @Override public String text() throws IOException {
        return Files.readString(this.path);
    }
}
```

---

## 5. Decoration & composition

Add behavior by **wrapping** an object (decorator/envelope), never by extending a concrete class
(`B5.1`, `B5.4`). A decorator implements the same interface, holds the wrapped object, and adds one
concern. This is the only legal reuse mechanism besides subtyping via interface extension (`B5.2`).

### Rules

- **[MUST]** Add behavior by wrapping, never by implementation inheritance (`B5.1`, `Ada.18`).
- **[MUST]** Classes are `final` (or `abstract`); an envelope base's delegate methods are `final`
  (`B5.3`, `B5.12`, `RsWrap.java:28`).
- **[MUST]** Inherit only to refine an `abstract` class, whose non-extension methods are `final`
  (`B5.2`, `Acoa.24`).
- **[MUST]** Replace `if`/`for`/`switch`/`while` with composable objects; move branch-if-you-run
  logic into a decorator (`B5.9`, `Acs.30`).
- **[MUST]** Never use static methods as reusable components; wrap a third-party utility in a small
  object (`B5.10`, `B5.11`).
- **[MUST]** Reject traits/mixins; layer decorators instead (`Acob.26`).
- **[MUST]** Avoid the Builder pattern; a growing argument list means extract collaborators, not a
  builder (`B5.8`, `B12.14`).
- **[SHOULD]** Validate with decorators, not inline checks (`Acs.16`, `Acs.21`).
- **[SHOULD]** Replace fluent chains with nested decorators; each concern is a new class, not a new
  method on a bloated one (`Acob.29`, `Acob.22`).

### Smells → fix

| Smell | Fix |
|---|---|
| `class Encrypted extends Document` | `Encrypted implements Document`, wraps `Document` (`B5.1`) |
| `if (log) { ... }` inside a method | `new Logged(origin)` decorator (`Acs.30`) |
| Fluent `request.method("GET").timeout(30)` | Nest decorated objects (`Acob.29`) |

### BEFORE / AFTER

```java
// BEFORE — implementation inheritance to add a concern
class Document {
    byte[] content() {
        return new byte[0];
    }
}
final class EncryptedDocument extends Document {
    @Override byte[] content() {
        return encrypt(super.content());
    }
}
```

```java
// AFTER — interface + decoration; each class `final`, one concern each
interface Document {
    byte[] content();
}
final class PlainDocument implements Document {
    @Override public byte[] content() {
        return new byte[0];
    }
}
final class EncryptedDocument implements Document {
    private final Document origin;
    EncryptedDocument(Document doc) {
        this.origin = doc;
    }
    @Override public byte[] content() {
        return encrypt(this.origin.content());
    }
}
```

---

## 6. Exceptions & null

`null` is banned across every public contract — never accept it, never return it, never signal with
it (`B6.1`, `B6.3`). **Fail fast** with checked exceptions (`B6.4`), catch only to chain and rethrow
(`B6.6`), and recover exactly once at the top-level entry point (`B6.8`).

### Rules

- **[MUST]** Never accept `null`; use a **Null Object** or a `Mask`-style object (`B6.1`, `B6.13`).
- **[MUST]** Never return `null`; choose: throw, return an empty collection, or a Null Object
  (`B6.3`).
- **[MUST]** Fail fast with checked exceptions; never return sentinel values (`-1`, `0`) or safe
  defaults (`B6.4`, `Atq.10`).
- **[MUST]** Throw only checked exceptions; one exception type is usually enough (`B6.5`, `B6.9`).
- **[MUST]** Don't catch unless to chain and rethrow; wrap the cause with context (`B6.6`, `B6.16`).
- **[MUST]** Recover exactly once, at the top-level entry point (`B6.8`).
- **[MUST]** Never catch-and-log; never use exceptions for flow control (`B6.10`, `B6.15`).
- **[MUST]** Avoid `java.util.Optional` as a null replacement (`B6.14`).
- **[SHOULD]** Put maximum context in every message, naming the concrete object/identifier
  (`Atq.11`).
- **[SHOULD]** A Null Object answers common calls benignly and throws only on entity-specific ones
  (`Acoa.26`, `Acoa.27`).
- **[NICE]** Inject the logger; default to a no-op `Log.NULL` (`Acob.33`).

### Smells → fix

| Smell | Fix |
|---|---|
| `return null;` | Throw, return empty collection, or Null Object (`B6.3`) |
| `catch (Exception ex) { log.error(ex); }` | Chain and rethrow, or delete the catch (`B6.10`) |
| `catch (...) { return -1; }` | Let it propagate; fail fast (`B6.4`) |

### BEFORE / AFTER

```java
// BEFORE — null signal, no context, silent failure
User user(String name) {
    final User found = this.db.find(name);
    if (found == null) {
        return null;
    }
    return found;
}
```

```java
// AFTER — fail fast with a checked, contextual exception
User user(String name) throws NotFoundException {
    final int id = this.db.id(name);
    if (id == 0) {
        throw new NotFoundException("No user: " + name);
    }
    return new StoredUser(this.db, id);
}
```

```java
// Null Object: benign for common calls, refuses impossible ones
final class NullUser implements User {
    @Override public String name() { return "anonymous"; }
    @Override public void raise(Cash s) { throw new IllegalStateException("stub"); }
}
```

---

## Self-check before commit

1. Every class is `final` or `abstract`; every field is `private final`.
2. No `null` accepted, returned, or used as a signal in any public contract.
3. Constructors are code-free: only assignments or `this(...)`/`super(...)`.
4. Exactly one primary constructor, declared last; all others delegate via `this(...)`.
5. No `new` outside secondary constructors; collaborators are constructor-injected.
6. No getters/setters, no `get`/`set` prefixes; behavior instead of data access.
7. No `-er`/`-or` class names and no `*Client` suffix; names say what the object *is*.
8. Every public method implements an interface method; interfaces are ≤3 methods.
9. No implementation inheritance; variant behavior is added by decoration/envelope.
10. No `public static` behavior, no utility class, no Singleton, no Builder.
11. `equals`/`hashCode` derive from encapsulated coordinates; no `instanceof` in business logic.
12. Exceptions are checked and contextual; no catch-and-log; recovery only at the top entry.
13. Mutating operations return a new object; no mutable internal data leaks out.
14. Every changed public interface has an updated fake/Null Object, plus a test asserting it.
15. `mvn --errors --batch-mode clean install -Pqulice` is green.
