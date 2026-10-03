# Elegant Objects — book postulates and examples

Source: `the "Elegant Objects" book by Yegor Bugayenko`, "Elegant Objects" by Yegor Bugayenko, Volume 1
(23 recommendations in four chapters: Birth / Education / Employment / Retirement).
Every postulate below is grounded in the OCR text; section 13 records coverage and OCR defects.
Code is adapted/trimmed from the book (the book itself uses pseudo-Java in places — e.g. its own
`Number`/`StringAsInteger`/`Millis` are invented). Anti-patterns are marked with `// wrong` / `// anti-pattern`.

## 1. Core OOP postulates

- **B1.1 A class is a factory of objects, not a template of functions** — Think of a class as a
  warehouse/mother that *makes objects*; when asked, it returns a live instance. It is not a passive
  listing of code that is copied when needed. Why: the "template of an object" definition makes the
  class brainless and hides the fact that objects are active entities. Objects are anthropomorphized
  ("he") and must be respected as self-sufficient representatives.
  Example (Java):
  ```java
  class Cash {
      private final int dollars;
      Cash(int dlr) { this.dollars = dlr; }   // factory: builds one object
  }
  Cash five = new Cash(5);
  ```

- **B1.2 An object is a representative, not a connector** — An object is not a pipe that passes data
  through untouched; it owns its encapsulated data and makes its own decisions. Why: a connector is
  not respected because it only moves information; a representative is a self-sufficient entity that
  acts on its own.
  Example (Java):
  ```java
  // wrong: a procedural "connector" that manipulates two arrays
  void find_prime_numbers(int[] origin, int[] primes) { /* ... */ }

  // right: an object that IS the filtered list
  class PrimeNumbers implements Iterable<Integer> {
      private final Iterable<Integer> origin;
      // iterates only primes, decides itself what to expose
  }
  ```

- **B1.3 An object represents a real-life entity** — Every object stands for something that exists
  outside the program's scope (a file on disk, a web page, a pixel, an SQL record). Its state is the
  *coordinates* of that entity, not free-floating data. Why: only entities that exist in the real
  world have coordinates (identity); objects without a represented entity degenerate into data bags.
  Example (Java):
  ```java
  class WebPage {                    // represents a page on the web
      private final URI uri;         // coordinates of the real entity
      WebPage(URI path) { this.uri = path; }
      String content() { /* HTTP GET */ return ""; }
  }
  ```

- **B1.4 A class must be either `final` or `abstract`, never neither** — Choose deliberately: `final`
  is a solid black box; `abstract` is an incomplete glass box meant to be refined. Why: a class that
  is neither may behave as solid while child classes override its virtual methods, causing
  counter-intuitive top-down/bottom-up behavior and unreadable hierarchies.
  Example (Java):
  ```java
  final class DefaultDocument implements Document { /* ... */ }
  abstract class Document { public abstract byte[] content(); public final int length() { return content().length; } }
  ```

- **B1.5 An object must encapsulate state, and that state is its identity** — Java separates identity
  from state (two equal objects with `==` are different shells); EO says state *is* identity.
  Encapsulate something; equality is defined by the encapsulated coordinates. Why: without state an
  object has no coordinates in the universe and cannot exist as a distinct entity.
  Example (Java):
  ```java
  class Cash {
      private final int dollars;
      private final int cents;
      Cash(int dlr, int cnt) { this.dollars = dlr; this.cents = cnt; }
      @Override public boolean equals(Object obj) { /* compare fields */ return true; }
      @Override public int hashCode() { return java.util.Objects.hash(dollars, cents); }
  }
  ```

- **B1.6 Encapsulate four objects or fewer** — More than four encapsulated objects is a refactoring
  signal. Why: identity is like coordinates in the universe; more than four coordinates is
  counter-intuitive. Large state should be grouped into sub-objects (a small tree), never a flat bag.
  Example (Java):
  ```java
  class Cash {                       // 3 coordinates
      private final Integer digits;
      private final Integer cents;
      private final String currency;
  }
  ```

- **B1.7 Encapsulate something at the very least** — A class with no properties is isomorphic to a
  static method and has no identity. Why: philosophically there is only one entity with no
  coordinates (the Universe); a stateless object cannot exist in pure OOP where `new` is confined to
  constructors.
  Example (Java):
  ```java
  // wrong: stateless behavior only
  class Year { int read() { return System.currentTimeMillis() > 0 ? 1 : 0; } }

  // right: encapsulate the dependency
  class Year { private final Millis millis; Year(Millis msec) { this.millis = msec; } }
  ```

- **B1.8 Every public method must implement an interface** — A class must have no public method that
  does not override some interface method. Why: interface-less public methods tightly couple callers
  to a concrete class and make mocking/decoration impossible. A class exists because someone needs
  its service; that service must be a documented contract.
  Example (Java):
  ```java
  interface Cash { Cash multiply(float factor); }

  final class DefaultCash implements Cash {
      private final int dollars;
      DefaultCash(int dlr) { this.dollars = dlr; }
      @Override public Cash multiply(float factor) { return new DefaultCash(this.dollars * (int) factor); }
  }
  ```

- **B1.9 Never use static methods — not even private ones** — Statics are procedural sub-routines in
  OOP syntax; they are un-mockable, un-decoratable, unhideable coupling. Why: statics cannot be passed
  as ctor arguments, break composition, encourage "thinking like a computer", and are global state.
  Example (Java):
  ```java
  // wrong
  class WebPage { public static String read(String uri) { /* HTTP */ return ""; } }
  String html = WebPage.read("http://www.java.com");

  // right
  class WebPage { private final String uri; WebPage(String u) { this.uri = u; } public String content() { /* HTTP */ return ""; } }
  String html = new WebPage("http://www.java.com").content();
  ```

- **B1.10 Never create utility classes (helpers)** — A "utility" class is a collection of static
  methods; it is not a factory of objects and therefore not a class. Why: it is an aggregation of all
  the static-method sins, a hard-coded, unbreakable dependency. A private ctor used to block
  instantiation is a smell, not a pattern.
  Example (Java):
  ```java
  // anti-pattern
  final class Math { private Math() {} public static int max(int a, int b) { return a < b ? b : a; } }
  // right: Max is an object implementing Number
  class Max implements Number { private final Number a; private final Number b; /* ... */ }
  ```

- **B1.11 Never use the Singleton Pattern** — A singleton is a global variable hidden behind a static
  `getInstance()`. Why: OOP has no global scope; the only difference from a utility class is that a
  singleton can be replaced, but it still abuses the paradigm. Instead, inject everything the object
  needs through its constructor.
  Example (Java):
  ```java
  // anti-pattern: global instance
  class User { private static User INSTANCE = new User(); static User getInstance() { return INSTANCE; } }

  // right: inject the current user
  class Report { private final User user; Report(User u) { this.user = u; } }
  ```

- **B1.12 Never name a class with an "-er" suffix** — No Manager, Controller, Helper, Handler,
  Writer, Reader, Converter, Validator, Router, Dispatcher, Observer, Listener, Sorter, Encoder,
  Decoder (nor "Util"/"Utils"). Name the class by *what it is*, not what it does. Why: an "-er" name
  says the thing is a collection of procedures manipulating data — a procedural mindset and evidence
  that the object is a connector, not a representative.
  Example (Java):
  ```java
  // wrong
  class CashFormatter { public String format() { /* ... */ return ""; } }
  // right
  class Cash { public String usd() { /* ... */ return ""; } }
  ```

- **B1.13 Tell, don't ask; a getter/setter class is a data structure, not an object** — Never expose
  naked data through `getX()`/`setX()`. Objects are black boxes; data structures are glass boxes.
  Why: getters/setters violate encapsulation and turn an active object into a passive bag of bytes,
  which drags the whole codebase back into imperative/procedural style.
  Example (Java):
  ```java
  // anti-pattern
  class Cash { private int dollars; public int getDollars() { return dollars; } public void setDollars(int v) { dollars = v; } }
  // right: behavior, not data access
  class Cash { private final int dollars; Cash(int d) { this.dollars = d; } public String usd() { return String.format("$ %d", this.dollars); } }
  ```

- **B1.14 Avoid type introspection, casting, and reflection** — Never use `instanceof`,
  `Class.cast()`, `((Type) obj)`, or reflection to branch on runtime type. Why: it discriminates
  objects by type (disrespectful), adds hidden coupling to more interfaces, and exposes undocumented
  expectations. Use polymorphism or method overloading instead.
  Example (Java):
  ```java
  // wrong
  public <T> int size(Iterable<T> items) {
      if (items instanceof java.util.Collection) { return ((java.util.Collection<T>) items).size(); }
      int size = 0; for (T item : items) { ++size; } return size;
  }
  // right: let the compiler choose the overload
  public <T> int size(java.util.Collection<T> items) { return items.size(); }
  public <T> int size(Iterable<T> items) { int size = 0; for (T item : items) { ++size; } return size; }
  ```

- **B1.15 Don't use public constants or `enum`** — Replace shared `public static final` literals and
  enums with micro-classes. Why: public constants introduce hard coupling and destroy cohesion — the
  constant is dumb and global, while a class can encapsulate the semantics of the value.
  Example (Java):
  ```java
  // wrong
  class Constants { public static final String CRLF = "\r\n"; }
  out.write(Constants.CRLF);

  // right: semantics isolated in a class
  class CRLFString {
      private final String origin;
      CRLFString(String src) { this.origin = src; }
      @Override public String toString() { return String.format("%s\r\n", origin); }
  }
  out.write(new CRLFString(rec.toString()));
  ```

- **B1.16 Prefer objects over primitives and naked data** — Never let data become more complex than a
  single byte in the hands of the caller; hide it inside objects. Why: naked data forces callers to
  write statements/operators manipulating bytes — imperative programming — and destroys the object
  model. More (small) classes make code *more* readable, like more words do in a language.
  Example (Java):
  ```java
  // wrong: primitive obsession / naked int
  Cash five = new Cash(5);
  int dollars = five.dollars;             // touching the data directly
  // right
  Cash five = new Cash(5);
  boolean enough = five.moreThan(new Cash(3)); // ask the object
  ```

- **B1.17 Compose smaller objects into bigger ones; stay declarative** — OOP is the job of composing
  bigger objects from smaller ones; code should *declare* what something is, not execute an algorithm.
  Why: declarative composition is faster (lazy/on-demand), decoupled (objects are first-class, can be
  swapped), more expressive, and free of temporal coupling.
  Example (Java):
  ```java
  // imperative // wrong
  Collection<Integer> evens = new LinkedList<>();
  for (int n : numbers) { if (n % 2 == 0) { evens.add(n); } }

  // declarative // right
  Collection<Integer> evens = new Filtered<>(numbers, number -> number % 2 == 0);
  ```

- **B1.18 Separate instantiation from execution** — First build the object graph, then hand it control
  (`new App(new Data(), new Screen())` then `app.run()`). Why: while building, nothing should happen;
  constructors only assemble. This keeps objects lazy, controllable and testable.
  Example (Java):
  ```java
  App app = new App(new Data(), new Screen());
  app.run();       // execution starts only here
  ```

- **B1.19 Don't use `new` outside secondary constructors** — The only legal place for `new` is a
  secondary constructor; methods and the primary ctor must receive their dependencies already
  constructed. Why: `new` inside a method hard-codes a dependency and makes the class untestable and
  unmaintainable. Ctor injection is the EO form of dependency injection / inversion of control.
  Example (Java):
  ```java
  // wrong: hard-coded dependency
  int euro() { return new Exchange().rate("USD", "EUR") * this.dollars; }
  // right
  class Cash {
      private final int dollars; private final Exchange exchange;
      Cash(int v, Exchange exch) { this.dollars = v; this.exchange = exch; }
      int euro() { return this.exchange.rate("USD", "EUR") * this.dollars; }
  }
  ```

- **B1.20 Expose fewer than five public (and protected) methods** — Count only public/protected
  methods (not ctors, not private) as the primary size metric; five or more demands refactoring. Why:
  small classes are more elegant, maintainable, cohesive (all methods use all properties) and
  testable.
  Example (Java):
  ```java
  final class WebPage {
      private final URI uri;
      WebPage(URI path) { this.uri = path; }
      String content() { /* ... */ return ""; }   // 1 public method
      void update(String content) { /* ... */ }   // 2
  }
  ```

- **B1.21 Keep every class under 250 lines of code** — In Java, 250 LOC (including comments and blank
  lines) is the maximum; aim lower; the same limit applies to test code. Why: short classes are
  understandable; a 1,000-line class is unreadable even to its author. Ruby's suggested limit is 100.
  Example (Java):
  ```java
  // If a class approaches 250 LOC, extract cohesive behavior into a new micro-class.
  ```

- **B1.22 All classes must be immutable** — No mutable objects may exist. If a value must change,
  produce a *new* object. Why: mutability is a procedural inheritance that causes identity-mutability
  bugs, non-atomic failures, temporal coupling, side effects and thread races — and it forbids pure
  OOP. See section 3.
  Example (Java):
  ```java
  // wrong
  class Cash { private int dollars; public void mul(int f) { this.dollars *= f; } }
  // right
  class Cash { private final int dollars; Cash(int d) { this.dollars = d; } public Cash mul(int f) { return new Cash(this.dollars * f); } }
  ```

- **B1.23 No `null` anywhere** — Never accept `null`, never return `null`, never use it to signal
  "unset". Why: `null` is a toxic keyword inherited from C pointers; it forces `== null` checks,
  destroys trust in objects, and encourages big multi-purpose classes. See section 6.
  Example (Java):
  ```java
  // wrong: lookup may fail -> null
  User user(String name) { /* ... */ return null; }
  // right: null object
  User user(String name) { /* ... */ return new NullUser(name); }
  ```

- **B1.24 A pure OOP language has no procedural operators** — `if`, `for`, `switch`, `while` are
  procedural; pure EO would provide `If`, `For`, `Switch`, `While` classes. Why: operators are a
  declarative-to-imperative leak; replacing them with objects keeps the code declarative and
  composable.
  Example (Java):
  ```java
  // imperative // wrong
  float rate; if (client.age() > 65) { rate = 2.5f; } else { rate = 3.0f; }
  // declarative // right
  float rate = new If(new GreaterThan(new AgeOf(client), 65), 2.5f, 3.0f);
  ```

- **B1.25 OOP beats FP for expression, but share FP's single-exit discipline** — Objects are more
  expressive than functions; avoid Java lambda-heavy FP drift. In an ideal OOP language methods would
  be true functions with a single exit point. Why: FP is a great paradigm but objects carry state,
  contracts and composition that functions do not. Prefer many small, composable, immutable objects.
  Example (Java):
  ```java
  // Prefer a named object that declares intent over an anonymous functional chain.
  final class Max implements Number {
      private final int a; private final int b;
      Max(int left, int right) { this.a = left; this.b = right; }
      @Override public int intValue() { return this.a > this.b ? this.a : this.b; }
  }
  ```


### Additions from articles

- **Acoa.1 Define an object as a representative/proxy, not a data container** -> add to §1:
  An object is a *representative* of a real-life entity that exists outside the program's scope; it
  does not *contain* data, it knows the entity's *coordinates* and animates it. Why: Java's memory
  model tempts us to call an object "a box with data", which provokes procedural access; the
  "representative" definition removes that temptation. Example (Java):
  ```java
  // HTTPStatus "points to" a page; it doesn't own the page's content.
  final class HTTPStatus implements Status {
    private URL page;
    public HTTPStatus(URL url) { this.page = url; }
    public int read() throws IOException {
      return HttpURLConnection.class.cast(
        this.page.openConnection()
      ).getResponseCode();
    }
  }
  ```

- **Acoa.2 Treat an immutable object as an *animator* of mutable data, not as dead data** -> add
  to §1: The role of an object is to make a piece of data alive without becoming that data; the
  data lives outside (file, HTTP, S3, memory) and the object merely holds the coordinates. Why:
  separates object *state* (identity) from the mutable *entity* it represents, so an immutable
  object can legitimately represent a mutable thing. Example (Java):
  ```java
  @Immutable
  final class Page {
    private final URI uri;                 // coordinates only
    Page(URI addr) { this.uri = addr; }
    public String load() {
      return new JdkRequest(this.uri).fetch().body();
    }
    public void save(String content) {
      new JdkRequest(this.uri).method("PUT").body().set(content).back().fetch();
    }
  }
  ```

- **Acoa.3 A class is where objects are born, not a "template of functions"** -> add to §1: A class's
  responsibility is to *construct* and *destruct* objects; the object asks the class to create
  another object and the class builds it — no third party "builds from a template". Why: the
  "template" framing puts classes in a passive position and hides that classes exist to make
  objects. Example:
  ```java
  // Ruby expresses it cleanly; `new` is the entry point to the class.
  photo = File.new('/tmp/photo.png')
  puts photo.width()
  ```

- **Acoa.4 Recognise that a pure-OOP method body is only `return` (and `new`)** -> add to §1: In a
  truly object-oriented world a method has a single `return` and *nothing else*; control flow
  (`if`, `>`, `for`) is expressed as objects (`If`, `GreaterThan`). Until Java gives us those
  classes, stick to a single `return`. Why: multiple `return`s and operators are procedural
  thinking leaked into OOP. Example (Java):
  ```java
  public int max(int a, int b) {
    return new If(new GreaterThan(a, b), a, b);
  }
  ```

- **Acoa.5 Ask "what real-life entity is behind this object? can I draw it?"** -> add to §1: If you
  cannot name/draw the real entity an object represents, refactor; controllers, parsers, filters,
  validators, service locators, singletons and factories represent nobody. Why: rename a parser
  to "parseable XML" and it suddenly represents a real artifact. No Java snippet needed.

- **Acoa.6 Prefer the smallest real entity an object can represent** -> add to §1: The more
  specific the entity, the more solid and cohesive the design; a no-arg constructor whose object
  can do everything represents "the Universe", and Universe objects are bad because there is only
  one Universe. Why: `new HTTP().read(url)` vs `new HTTP(url).read()` — the constructor's
  encapsulated arguments define the entity. Example (Java):
  ```java
  new HTTP("https://www.google.com").read(); // represents a web page
  new HTTP().read("https://www.google.com"); // represents the Universe
  ```

- **Acob.1 Decompose responsibility vertically, not horizontally** -> add to §1: when an object does too much, extract the extra behavior into a *decorator* that wraps the original object, not into a sibling class the client must combine manually. Why: horizontal decomposition (client holds both `Log` and `Line`) adds dependencies and contact points, while vertical decomposition (a `TimedLog` wrapping `Log`) keeps a single entry point and lowers complexity.
  ```java
  final class TimedLog implements Log {
    private final Log origin;
    TimedLog(Log log) { this.origin = log; }
    @Override public void put(String text) {
      this.origin.put(new Timestamp().plus(text));
    }
  }
  // usage: Log log = new TimedLog(new FileLog("/tmp/log.txt"));
  //        log.put("Hello, world");   // still one contact point
  ```

- **Acob.2 Treat types as vocabulary; many small classes are a virtue** -> add to §1: a rich set of small, noun-named classes makes code more expressive and readable, so "too many classes" must not be treated as a drawback. Why: the author compares a large vocabulary ("Read the book on the table") to a tiny one ("Do it with the thing on that thing"); rules like no static methods, ≤4 attributes, code-free ctors, ≤5 public methods inevitably produce many classes — that is the goal.
  ```java
  // Instead of one BookDTO with 50 fields/methods, prefer:
  interface Book { String isbn(); }
  final class JsonBook implements Book { /* ... */ }
  final class CachedBook implements Book { /* ... */ }
  ```

- **Acob.3 Reject MVC-style "controller in charge"** -> add to §1: MVC is procedural because a controller pulls naked data out of the model, decides its meaning, and pushes it into a view; the OO alternative is objects decorating the same real entity. Why: in `int s = load_from_engine(); printf("The speed is %d mph", s);` only the controller knows the unit (mph); each new client must re-assume it.
  ```java
  // OO alternative: same entity, grown by decoration, never torn apart
  new PrintedSpeed(new FormattedSpeed(new SpeedFromEngine())).toString();
  ```

- **Acob.4 Objects are alive; implementation inheritance kills them** -> add to §1: `extends` that copies methods/fields from a parent is a procedural code-reuse technique, not OOP; allow only *subtyping* (`interface Article extends Manuscript`) which enables polymorphism and LSP. Why: "inherit" as copy-from-a-dead-parent treats an object as dead property; an object that lets others inherit its code is "dead". Subtyping derives a characteristic (an `Article` *is a* `Manuscript`), which is correct.
  ```java
  interface Manuscript { void print(Console c); }
  interface Article extends Manuscript { void submit(Conference c); } // subtyping, good
  ```

- **Acob.5 Design existence as its own quality, not merged into another** -> add to §1: put `exists()` on the object (after it is constructed) rather than on a static `Disk.fileExists()`; the two failures ("I don't exist" and "I'm not paid") are distinct and deserve distinct handling. Why: `bills.paid(42)` merges existence and payment into one message, hiding a nullable column; `Bill b = bills.get(42); if (b.paid())` preserves both qualities and two points of failure.
  ```java
  File f = new File("a.txt");   // may fail: bad/NULL name
  boolean e = f.exists();       // may fail: unmounted disk / permissions
  ```

- **Acob.6 An object may have no methods at all** -> add to §1: in EO an object is a tree of attributes that are other objects; touching an attribute (not calling a method) triggers computation lazily. Why: methods are procedures inherited from C/ALGOL; EO replaces method calls with attribute access and defers work until the body `𝜑` is touched.
  ```text
  [r] > circle
    mul 2 3.14 r > perimeter
    mul 3.14 r r > area
  circle 30 > c
  c.area > a          # not a call; takes an already-built atom
  ```

- **Acob.7 Abstract objects hold free attributes and are completed by application** -> add to §1: model templates as objects with "free" attributes; copying with arguments (application) yields a closed object, and free attributes may not be touched. Why: EO's `book` with free `id`/`db` cannot be `book.title`-ed until applied; partial application with `:db` leaves it abstract on purpose.
  ```text
  [id db] > book
    db.query > title
      "SELECT title FROM book WHERE id=?"
      id
  book 42 mysql > b     # closed object
  ```

- **Atq.1 Treat debugging as evidence of bad design** -> add to §1: If you feel the need to step through code in a debugger, the design is wrong; replace debugging with a unit test that reproduces the problem. Why: debugging finds a problem once but cannot prevent it from recurring, while a test is a permanent investment. Example (Java):
  ```java
  // Bad: static procedural method that you must debug
  class FileUtils {
    public static Iterable<String> readWords(File f) { ... }
  }

  // Good: small noun-object, trivially testable
  final class Words implements Iterable<String> {
    private final String text;
    @Override
    public Iterator<String> iterator() {
      final Set<String> words = new HashSet<>();
      for (final String word : this.text.split(" ")) {
        words.add(word);
      }
      return words.iterator();
    }
  }
  ```

- **Atq.2 Judge a bug by requirements, not only by behavior** -> add to §1: A defect is any violation of a functional OR non-functional requirement (maintainability, reusability, documentation, style); "it behaves as intended" is not proof of no bug. Why: non-functional flaws are often more expensive to fix than functional ones. Example: an unmaintainable PDF generator (hard-coded A4, no extension point) is a bug even while it prints correct PDFs.

- **Adt.1 Treat IoC as object composition, not a DI container** -> add to §1: Inversion of control means handing a whole object to a collaborator so *it* asks the questions (`print(book)`), not a container that injects data into fields. Why: the container removes `new` from the code, hides composition, and the object loses control over itself (annotations are "a big mistake" for the same reason). Example:
  ```java
  // wrong: control (and data) leaked out
  print(book.title());
  // right: delegate — or better, replace the procedure with an object
  print(book);
  new PrintedBook(book);
  ```

- **Adt.2 Delete annotations that carry behavior** -> add to §1: Do not implement object functionality outside the object via `@Inject`, `@XmlElement`, `@RetryOnFailure`; put the behavior in a decorator or in the class itself and compose explicitly with `new`. Why: AOP/annotation magic keeps half the object elsewhere, breaks encapsulation, and hides the composition step we must be able to see. Example:
  ```java
  Foo foo = new FooThatRetries(new Foo());     // visible composition
  String xml = new XmlBook(new DefaultBook("Elegant Objects")).toXML();
  ```

- **Adt.3 Distinguish builders from manipulators** -> add to §1: A method that returns something is a *builder* and must be a noun (`book()`, `salary()`); a method that changes the world is a *manipulator*, a verb, and returns `void` (`add()`, `save()`). Why: it separates declarative composition from imperative instructions, removes side effects from queries, and yields shorter, clearer names.
  ```java
  interface Bookshelf {
    Book book(String title);   // builder, noun, no side effect
    void add(Book book);       // manipulator, verb, void
  }
  ```

- **Adt.4 Avoid static factory methods; instantiate with constructors** -> add to §1: Use `new ListOf(1, 2, 3)` rather than `List.of(1, 2, 3)`, and provide constructors instead of static factories / utility classes. Why: `new` makes the instantiated class and its real constructor arguments visible; static methods turn classes into collections of functions and make the moment of object birth invisible.
  ```java
  List<Integer> list = new ListOf(1, 2, 3);   // Cactoos: OOP
  ```

- **Adt.5 Keep interfaces functionality-poor** -> add to §1: Design an interface around the single responsibility it exposes (`InputStream` = `read(byte[],int,int)`), and push convenience like single-byte reading into a "smart" decorator (`InputStream.Smart`). Why: overloaded methods bloat types, and adding `transferTo()`/`readAllBytes()` to the JDK stream is a textbook ISP violation. More than three methods or overloaded methods is a smell.

- **Ada.1 Compose the application with `new`, not a DI container** -> add to §1: An object gets its dependencies as constructor arguments supplied by plain object composition at the application edge (`main`), never resolved by a DI container. Why: field/setter injection produces incomplete mutable objects, while even "constructor injection" through a container only adds files, annotations and indirection without adding behavior — the wiring you can already express with `new`. Example (Java):
  ```java
  public final class App {
    public static void main(final String... args) {
      final Budget budget = new Budget(
        new Postgres("jdbc:postgresql:5740/main")
      );
      System.out.println("Total is: " + budget.total());
    }
  }
  ```

- **Ada.2 Don't follow SOLID; it is procedural OOP for dummies** -> add to §1: Treat SOLID as a marketable paraphrase of Constantine's 1974 cohesion/coupling, not as an object design method — reject "O" because it blesses implementation inheritance, and recognize that "S", "I", "D" merely restate high cohesion and loose coupling. Why: the principles are vague ("one reason to change"), and OCP literally pushes an anti-OOP technique; relying on them prevents understanding the real mechanics. Example (Java):
  ```java
  // SOLID 'D' is just loose coupling: depend on the interface,
  // not the implementation — a language feature, not a principle.
  final List<String> items = new ArrayList<>(); // not ArrayList<String>
  ```

- **Ada.3 Encapsulation comes first, size goes next** -> add to §1: Judge a class by how tightly it protects what it encapsulates, not by how many "responsibilities" it has; a large cohesive object is preferable to a DTO. Why: applying SRP to its full extent turns an object into a bare holder of a hidden resource (getter + a few procedures), destroying decorability and true OO. Example (Java):
  ```java
  // Better: an object that guards its AWS client and knows how to read itself.
  final class AwsOcket {
    boolean exists() { /* ... */ }
    void read(final OutputStream output) { /* ... */ }
  }
  // Worse (SRP taken to the limit): exposes the client, becomes a carrier.
  // new ContentReader(ocket.aws()).read(System.out);
  ```

- **Ada.4 Keep data and its processor in the same object** -> add to §1: Do not convert a request into a DTO, hand the data to a controller, and get a response DTO back; the object that holds the data must be the object that acts on it. Why: separating data from behavior is the procedural path taken by Spring-style frameworks and produces passive data bags instead of objects. Example (Java):
  ```java
  interface Resource {
    Resource refine(String name, String value);
    void print(Output output);
  }
  ```

- **Ada.5 Prefer composition over messaging between peer modules** -> add to §1: When two objects must interact, create a bigger object that encapsulates the smaller ones and lets them interact inside, rather than sending data messages between equal-level objects. Why: messaging keeps both objects at the same abstraction level and forces data exposure (getters/printers), so the maintainability problem is never solved. Example (Java):
  ```java
  // messaging (procedural, peers):
  // point.printTo(canvas);
  // composition (an encapsulating object):
  final Object printed = new PrintedOn(point, canvas);
  ```

- **Ada.6 Let the compiler infer types instead of declaring them** -> add to §1: The information needed to type an incoming argument is already in the method body; explicit interfaces should be inferred from the messages the object receives. Why: strong typing prevents errors but costs type declarations and casting; if casting is prohibited, the compiler can derive the type from usage. Example (Java):
  ```java
  // b.isbn() proves b needs at least an isbn() method;
  // the type Book can be inferred, not declared.
  void print(Book b) {
    System.out.printf("The ISBN is: %s%n", b.isbn());
  }
  ```

- **Ada.7 Treat objects as things that dataize into data** -> add to §1: In EO, an object is composed via abstraction (declaring an abstract object) or application (making a copy with arguments), and only *dataization* turns it into the platform's primitive data. Why: numbers and strings are objects too; the raw data lives inside atoms and is only produced when the runtime asks the object to become data. Example (EO/Java bridge):
  ```java
  import org.eolang.phi.Data;
  EOapp app = new EOapp();
  Boolean data = new Dataized(app).take(Boolean.class);
  ```

- **Acs.1 Expose behavior, never data** -> add to §1: A class must encapsulate all of its data so that no value can escape it, directly or through a getter; only functionality (methods that produce new results) is public. Why: any data element that escapes an object is "naked" and creates hidden coupling, because surrounding code makes unstated assumptions about it (units, encoding, precision). Example (Java):
  ```java
  final class Temperature {
    private final int t;
    Temperature(int value) { this.t = value; }
    public String toString() { return String.format("%d F", this.t); }
  }
  ```

- **Acs.2 Ban globally-visible state** -> add to §1: Never use global variables, singletons, class variables, or "globally scoped" static configuration; make every collaborator an injected object so many differently-configured instances can exist at once. Why: globals destroy composability — they force a single application-wide instance and make it impossible to build alternative configurations (e.g. two servers in one test). Example (Java):
  ```java
  final class App {
    private final int port;
    App(int p) { this.port = p; }
    void start() { /* ... */ }
  }
  new App(8080).start();
  new App(9090).start();
  ```

- **Apa.1 Design a language where everything is an object and `byte`/`bytes` are the only built-ins** -> add to §1: Treat all values as objects and avoid scalar/primitive types as the fundamental abstraction. Why: scalar types, `null`, statics and reflection are procedural leftovers that force type/behavior decisions out of the object. Example:
  ```text
  Principles: everything is an object; byte/bytes are the only built-in types;
  strict compile-time static analysis.
  Forbidden: no mutable objects, no public/protected properties, no static members,
  no global vars, no enums, no NULL, no unchecked exceptions, no interface-less
  classes, no implementation inheritance, no instanceof, no reflection,
  all methods final or abstract, no root Object class.
  ```

- **Apa.2 Treat compile-time static analysis as a first-class language feature, not a tool** -> add to §1: Make strict static analysis part of the definition of the language/build, so violations cannot even compile. Why: quality enforced by the platform is non-optional, unlike a volunteer report. Example:
  ```text
  "strict compile-time static analysis" listed beside "everything is an object"
  as a key principle of the language.
  ```

- **Apa.3 Model the world, not instructions: prefer declarative over imperative design** -> add to §1: A good object (like a good manager) declares *what* is expected and lets others decide *how*; avoid embedded algorithms. Why: imperative style hard-codes the way and strangles the behavior, exactly like micromanagement. Example:
  ```text
  Good manager: "The server with Nginx must be up by 6 p.m."
  Micromanager: "Install Nginx now and don't do anything else until done."
  ```

- **Apa.4 Reject anti-patterns early because they metastasize** -> add to §1: A single anti-pattern (God object, static utility, Singleton) is a tumor; once admitted it multiplies until the software must be rewritten. Why: flexibility of languages (`Java` lets you put 1000 methods in one class) must be constrained by discipline. Example:
  ```text
  // NO God object; NO Singleton; NO utility class; NO global mutable state
  ```

- **Apb.1 Shrink the scope of visibility** -> add to §1: Every construct must minimize the scope in
  which its data and helpers are visible; prefer a three-line `for`-scope over a shared variable that
  lives across a whole method. Why: maintainability is "the time required to understand the code," and
  the larger the visible scope of a variable, the longer that time becomes — every reader must trace
  the full lifetime of the data. Example:
  ```text
  // worse: one `i` visible across 10 lines
  int i = 0; while (++i < 10) {...} i -= 10; while (++i < 10) {...}
  // better: two independent 3-line scopes
  for (int i = 0; i < 10; ++i) {...}
  for (int i = 0; i < 10; ++i) {...}
  ```

- **Apb.2 Objects exist to force small scopes** -> add to §1: Use objects to make large scopes
  *impossible*, not merely discouraged — the object must hide its data so callers cannot build logic
  on it. Why: a class that only exposes getters/setters is a data holder; the real logic stays
  outside, the scope does not shrink, and the reader now has two things to understand (the caller and
  the holder). Example:
  ```java
  // A proper Line: caller can only ask it to move and print, never read its state
  final class Line {
    private final int mul; private int v = 0;
    void print() { /* prints v*mul */ }
    boolean next() { /* advances, returns false at end */ }
  }
  ```

- **Apb.3 Prefer aesthetics over functionality** -> add to §1: Treat ugly-but-working code as a
  defect to fix, not as an acceptable minimum; elegance (modularity, naming, cohesion, error handling)
  is a first-class requirement. Why: "elegantly designed software is easier to fix than working
  software is to make elegant," and functionality-only code cannot be retrofitted into clean design
  later without a rewrite. Example:
  ```text
  // A working method with swallowed exceptions, NULL, mutable fields and a 10-arg signature
  // is not "done" — it is a defect list to refactor, even though tests pass.
  ```

### Additions from real codebases

Mapped from `knowledge/elegant-objects-java.md`, `knowledge/elegant-objects-java.md`, `knowledge/elegant-objects-java.md`, `knowledge/elegant-objects-java.md`,
`knowledge/elegant-objects-java.md`, `knowledge/elegant-objects-java.md`, `knowledge/elegant-objects-java.md`.

- **"Objects over statics" is provable at library scale.** Cactoos has **0 `public static`
  methods** across ~23,759 LOC in `src/main/java`, while shipping classes that replace JDK/Guava
  statics — `FormattedText` for `String.format`, `Lowered` for `toLowerCase`, `LengthOf`, `Filtered`,
  `Mapped` (`README.md:333-358`); it also has **zero runtime dependencies** (`pom.xml:68-115`,
  `README.md:32,414-419`). Takes likewise has 0 `public static` methods (only `public static final`
  constants: `Token.java:56,61,115,120,125`, `SiHmac.java:32,37,42`), 297 `final class` / 32
  interfaces / 0 `instanceof`.
- **The public abstraction is a one-method `@FunctionalInterface` at the package root.** Cactoos:
  `Scalar<T>` (`src/main/java/org/cactoos/Scalar.java:31-40`), `Text`
  (`src/main/java/org/cactoos/Text.java:19-28`), 11 interfaces / 10 `@FunctionalInterface`. Takes:
  `Take` (`src/main/java/org/takes/Take.java:47-58`). requs: `Step`, `Rule`, `Facet`, `XeFacet`
  (`Step.java:12`, `Rule.java:16`); note `Step extends Mentioned, Signature` so it *cannot* carry
  `@FunctionalInterface` (`knowledge/elegant-objects-java.md` §H).
- **Qulice encodes principle 6 as build gates.** `ProhibitPublicStaticMethods` (exempts
  `main(String[])`, `@BeforeClass`/`@AfterClass`/`@Parameters`), `AvoidDirectAccessToStaticFields`,
  `AvoidAccessToStaticMembersViaThis`, `StaticAccessViaInstanceCheck`, `NonStaticMethodCheck`,
  `ProhibitStaticNestedClassesCheck` (`ruleset.xml:746-810`, `checks.xml:489,499-503`).
- **The platform forces some statics — scope the rule to the public API.** JUnit's `@MethodSource`
  requires a `private static` provider and `@SafeVarargs` markers exist (Cactoos); `Entry.main` is
  `public static` and `Entry.pulse()` private static (rultor `Entry.java:78,179`); xembly's `Verbs`
  is a static parser over static mutable maps (`Verbs.java:30-56`). The workable rule is **no public
  static behaviour**, not "no `static` token anywhere".
- **Naming proof.** Neither Cactoos nor Takes has an `-er` package or class; Takes uses entity
  prefixes (`Tk`, `Rs`, `Rq`, `Fk`, `Bk`) and the longest class name is `RqWithDefaultHeader`. Older
  repos still carry `Verbs`, `Compiler`, `OptionParser`, `DirectoryListing` — flag these as
  migration targets, not templates (`knowledge/elegant-objects-java.md` §F, `knowledge/elegant-objects-java.md` §F).
- **Architecture can be enforced mechanically.** eo runs ArchUnit to check package rules and
  single-parent hierarchies (`eo-maven-plugin/src/test/java/org/eolang/maven/ArchitectureTest.java:31-60`);
  Mockito usage is ≈0 in large suites (eo, Cactoos ≈1 hit) because interfaces + real fakes replace it.

## 2. Constructors and object lifecycle

- **B2.1 Keep constructors code-free** — A constructor body must contain only assignments
  (in C++ it is empty). Do not parse, convert, validate, or otherwise "do work" in a ctor. Why:
  code-free ctors are lazy and controllable — the work happens only when a method is called, letting
  users avoid unnecessary computation and optimize deliberately.
  Example (Java):
  ```java
  // wrong: parsing in the ctor
  class StringAsInteger implements Number {
      private final int num;
      StringAsInteger(String txt) { this.num = Integer.parseInt(txt); } // wrong
      public int intValue() { return this.num; }
  }
  // right: wrap the argument, convert on demand
  class StringAsInteger implements Number {
      private final String source;
      StringAsInteger(String src) { this.source = src; }
      public int intValue() { return Integer.parseInt(this.source); } // deferred
  }
  ```

- **B2.2 Make one constructor primary** — Exactly one ctor initializes the encapsulated properties;
  all others are secondary. Why: a single initialization point avoids duplicated validation and
  conversion logic and yields cleaner, more maintainable code.
  Example (Java):
  ```java
  class Cash {
      private final Number dollars;
      Cash(float dlr) { this((int) dlr); }              // secondary
      Cash(String dlr) { this(Cash.parse(dlr)); }       // secondary
      Cash(Number dlr) { this.dollars = dlr; }          // primary (last)
  }
  ```

- **B2.3 Secondary ctors must delegate to the primary via `this(...)`** — Secondary ctors only
  prepare/convert/reformat arguments, then call the primary. Why: putting initialization in one place
  means a new rule (e.g. "amount must be positive") is added once, not in every ctor.
  Example (Java):
  ```java
  // wrong: three initialization sites
  Cash(float dlr) { this.dollars = (int) dlr; }
  Cash(String dlr) { this.dollars = Cash.parse(dlr); }
  Cash(int dlr) { this.dollars = dlr; }

  // right
  Cash(float dlr) { this((int) dlr); }
  Cash(String dlr) { this(Cash.parse(dlr)); }
  Cash(int dlr) { this.dollars = dlr; }
  ```

- **B2.4 Place the primary ctor last in the file** — After all secondary ctors. Why: when you reopen
  a class with ten ctors months later, you scroll to the last one instead of reading all of them to
  find where state is initialized.

- **B2.5 Don't touch the arguments in a constructor; wrap them** — A ctor may not call methods on its
  arguments or transform them; it should wrap or store them raw. Why: when a ctor is invoked, the
  object has not been asked to work yet; doing work there violates the instantiation/execution
  separation and hides side effects.
  Example (Java):
  ```java
  class Cash {
      private final Number dollars;
      Cash(String dlr) { this(new StringAsInteger(dlr)); }  // wrap, don't parse
      Cash(Number dlr) { this.dollars = dlr; }
  }
  ```

- **B2.6 Prefer many ctors over many methods** — A good class has a few methods and 5–10 ctors. Why:
  ctors give callers flexibility (build `Cash` from int, float, String, ISO text) while public methods
  add responsibilities and reduce cohesion.
  Example (Java):
  ```java
  new Cash(30);
  new Cash("$29.95");
  new Cash(29.95d);
  new Cash(29.95f);
  new Cash(29.95, "USD");
  ```

- **B2.7 Defer work to methods (lazy objects)** — Conversion, I/O and other expensive work belongs in
  the method that needs the result, not the ctor. Why: this avoids doing work that may never be used
  and keeps objects transparent and controllable.
  Example (Java):
  ```java
  Number five = new StringAsInteger("5");
  if (/* something is wrong */) { throw new IllegalStateException("problem"); }
  five.intValue();       // parsing happens only here
  ```

- **B2.8 Add a caching decorator when repeat computation matters** — If deferred parsing runs too
  often, wrap the object in a caching decorator rather than moving work back into the ctor. Why: you
  keep immutability and lazy behavior while controlling optimization at the call site.
  Example (Java):
  ```java
  class CachedNumber implements Number {
      private final Number origin;
      private final Collection<Integer> cached = new ArrayList<>(1);
      CachedNumber(Number num) { this.origin = num; }
      public int intValue() {
          if (this.cached.isEmpty()) { this.cached.add(this.origin.intValue()); }
          return this.cached.get(0);
      }
  }
  Number num = new CachedNumber(new StringAsInteger("123"));
  num.intValue(); // parses
  num.intValue(); // no parsing
  ```

- **B2.9 `new` is allowed only in secondary constructors** — Methods and the primary ctor must never
  call `new`. Why: this removes hard-coded dependencies and makes every object fully injectable,
  mockable (with fakes) and testable. When a method genuinely must create objects, extract a
  factory/envelope and inject it.
  Example (Java):
  ```java
  // wrong
  class Requests {
      private final Socket socket;
      Request next() { return new SimpleRequest(/* read */ ""); } // new in method
  }
  // right: inject a Mapping that does the construction
  class Requests {
      private final Socket socket; private final Mapping<String, Request> mapping;
      Requests(Socket skt) { this(skt, data -> new SimpleRequest(data)); } // new only in secondary ctor
      Requests(Socket skt, Mapping<String, Request> mpg) { this.socket = skt; this.mapping = mpg; }
      Request next() { return this.mapping.map(/* read */ ""); }
  }
  ```

- **B2.10 A secondary ctor may inject sensible default dependencies** — Convenience ctors may call
  the primary with a default implementation (e.g. `new NYSE()`), but the primary must allow full
  control. Why: callers get flexibility without losing testability.
  Example (Java):
  ```java
  class Cash {
      private final int dollars; private final Exchange exchange;
      Cash() { this(0); }
      Cash(int value) { this(value, new NYSE()); }        // default dependency
      Cash(int value, Exchange exch) { this.dollars = value; this.exchange = exch; }
  }
  ```

- **B2.11 A private ctor on a "utility class" is a smell, not a feature** — The private ctor exists
  only to prevent instantiation of a non-class; do not copy this pattern. Why: it confirms the type is
  not a factory of objects and is therefore not an object at all.
  Example (Java):
  ```java
  // anti-pattern
  class Math { private Math() {} public static int max(int a, int b) { return a < b ? b : a; } }
  ```

- **B2.12 Constructors are the software; statements are not** — In OOP the application is the graph
  of objects assembled by constructors, not a top-to-bottom script. Why: once you accept the object
  model, initialization through ctors *is* the program; the code is a secondary element.
  Example (Java):
  ```java
  App app = new App(new Data(), new Screen());
  app.run();
  ```

- **B2.13 Use ctor overloading (Java) or argument maps (unoverloaded languages)** — Method/ctor
  overloading is a fundamental OOP feature that makes code read like business language; where absent
  (Ruby, PHP), use maps of named arguments but still initialize in one place. Why: `content(File)` and
  `content(File, Charset)` read better than `content` and `contentInCharset`, and still converge on a
  single primary ctor.
  Example (Java):
  ```java
  class Cash {
      Cash(int dollars) { this.dollars = dollars; }
      Cash(java.util.Map<String, Object> args) { this((Integer) args.get("int")); } // fallback style
  }
  ```

- **B2.14 Instantiate, don't validate, in the primary ctor** — Validation belongs in behavior, not
  ctor bodies (a consequence of code-free ctors); the argument set of the primary ctor is complete by
  definition. Why: initialization is pure assembly; adding validation there reintroduces duplication
  and eager work. *(Book note: it suggests the primary ctor receives everything needed and wraps it —
  validation, if any, is a behavior method.)*
  Example (Java):
  ```java
  class Cash {
      private final Number dollars;
      Cash(Number dlr) { this.dollars = dlr; }               // no validation
      boolean positive() { return this.dollars.intValue() > 0; } // behavior
  }
  ```

- **B2.15 An immutable object is complete and solid after construction** — There must be no "skeleton
  first, setters later" lifecycle. Why: the two-step bean lifecycle creates temporal coupling and
  NULL-filled holes; a single ctor statement removes both.
  Example (Java):
  ```java
  // wrong: skeleton + setters
  Cash price = new Cash(); price.setDollars(29); price.setCents(95);
  // right: one statement, complete object
  Cash price = new Cash(29, 95);
  ```


### Additions from articles

- **Acoa.7 Classify constructors as primary and secondary; keep exactly one primary, declared last** ->
  add to §2: A secondary constructor does nothing but delegate via `this(...)`; the primary is the
  single construction entry point that assigns fields. Why: eliminates duplicated assignment logic
  across overloads and makes the "real" constructor easy to find (it is always the bottom one).
  Example (Java):
  ```java
  final class Cash {
    private final int cents;
    private final String currency;
    public Cash() { this(0); }                 // secondary
    public Cash(int cts) { this(cts, "USD"); } // secondary
    public Cash(int cts, String crn) {         // primary
      this.cents = cts;
      this.currency = crn;
    }
  }
  ```
  *(also Adt.7, Ada.10)*

- **Acoa.8 Let constructors only assign; move computation into methods/decorators** -> add to §2:
  Any computation in a constructor is an unrequested side effect and prevents composition
  (`new EnglishName(new NameInPostgreSQL(...))` should not hit the DB just to store a name). Why:
  the *ing* form computes eagerly, like a static method; defer it and add a `CachedName` decorator
  only if repeated calls hurt. Example (Java):
  ```java
  public final class EnglishName implements Name {
    private final CharSequence text;
    public EnglishName(final CharSequence txt) { this.text = txt; } // assignment only
    public String first() { return this.text.toString().split(" ", 2)[0]; }
  }
  ```
  *(also Adt.6, Ada.9, Apa.5)*

- **Acoa.9 Use the constructor's arguments to document the represented entity** -> add to §2: The
  arguments a constructor encapsulates identify the real-world entity the object will access; a
  no-arg constructor (with only a supplementary overload that assigns) means the object represents
  everything. Why: reading a constructor tells you what the object *is*. Example (Java):
  ```java
  class Time {
    private final long msec;
    public Time() { this(System.currentTimeMillis()); } // secondary
    public Time(long time) { this.msec = time; }        // primary — represents a moment
  }
  ```

- **Acob.8 Use constructors, never static factory methods** -> add to §2: names, caching, and subtyping are only excuses to fix a bad design; use polymorphism/encapsulation and real constructors. Why: `Color.makeFromPalette(...)`, static caches, and `Color.make(h)` forking all tear the decision logic out of the object it belongs to.
  ```java
  interface Color { }
  final class HexColor implements Color { HexColor(int h) { this.hex = h; } }
  final class RGBColor implements Color {
    RGBColor(int red, int green, int blue) { this(new HexColor(red << 16 + green << 8 + blue)); }
  }
  Color tomato = new RGBColor(255, 99, 71);
  ```

- **Acob.9 Move instance caching into a dedicated container object** -> add to §2: instead of a private static `Map` inside the class, make a `Palette`-like store object and hold it in the client. Why: `Color.makeFromPalette()` needs a static `CACHE` attribute; `new Palette().take(255,99,71)` achieves the same without statics and stays replaceable.
  ```java
  final class Palette {
    private final Map<Integer, Color> colors = new HashMap<>();
    Color take(int red, int green, int blue) { /* computeIfAbsent */ }
  }
  ```

- **Acob.10 Extract argument pre-processing into a prestructor** -> add to §2: keep exactly one primary constructor and one line in every secondary constructor; move any list/array building into a "prestructor" method or class. Why: `Books(String... array)` with a loop is code in a ctor and a second primary; `this(Books.toList(array))` (or `this(new ToList(array))`) fixes both.
  ```java
  final class Books {
    private final List<String> titles;
    Books(List<String> list) { this.titles = Collections.unmodifiableList(list); }
    Books(String... array) { this(new ListOf<String>(array)); } // ListOf from Cactoos
  }
  ```

- **Acob.11 Validate representable state in the ctor; runtime state in methods** -> add to §2: only reject arguments that the object cannot *represent* (null, wrong type) in the constructor; defer checks that depend on the world (file exists, is a file) to the behavior method. Why: the author distinguishes "connecting" (encapsulation) from "talking" (delegation); the file may appear seconds later, so delegating the existence check is correct and avoids temporal coupling.
  ```java
  final class Users {
    private final Path file;
    Users(Path file) {            // can only check state
      if (file == null) throw new IllegalArgumentException("null");
      this.file = file;
    }
    Iterable<String> names() {    // can check runtime conditions
      if (!Files.exists(this.file)) throw new IllegalStateException("absent");
      return new ListOf<>(Files.readAllLines(this.file));
    }
  }
  ```

- **Acob.12 Use "decorating envelopes" for ctor-only wrapper classes** -> add to §2: when a class exists only to build a composition of decorators, give it a single constructor and no new methods; it is an envelope, not a full object. Why: `RsHtml` adds no behavior, only composes `RsWithType(new RsWithStatus(...))`; the alternative is a verbose pass-through `RsWrap` or a static factory (avoided).
  ```java
  final class RsHtml implements Response {
    RsHtml(String text) { this(new RsWithType(new RsWithStatus(text, 200), "text/html")); }
    RsHtml(Response res) { this.origin = res; }
  }
  ```

- **Acob.13 Create similar objects by copying, not instantiating** -> add to §2: in a class-less OO model, build an object from an existing one via `copy` with different arguments; libraries hand you objects you copy. Why: the author's prototype has only types and objects — `Book b2 = copy b1("Elegant Objects")` — no implementation inheritance and no static methods.
  ```java
  // conceptual; Java best-effort: a "copy" is a decorated/cloned variant
  Book b2 = b1.withTitle("Elegant Objects");
  ```

- **Atq.3 Keep ctor work out of retry/aspect concerns** -> add to §2: Never put retry/recovery loops in a constructor; expose behavior on a method and let a decorator/aspect retry it. Why: a retried object must be safely reusable, and ctor behavior cannot be controlled or intercepted cleanly. Example (Java):
  ```java
  @RetryOnFailure(attempts = 3, delay = 10, unit = TimeUnit.SECONDS)
  public String load(URL url) {
    return url.openConnection().getContent();
  }
  ```

- **Adt.8 Own resource lifetime via try-with-resources (RAII)** -> add to §2: Wrap a resource (semaphore permit, connection, lock) in a `Closeable` object acquired in a `try (...)` header so release happens on normal, `return`, and `throw` paths. Why: Java has no deterministic destructor; `finalize()` is too late and release-before-every-throw duplicates code and leaks. Example:
  ```java
  try (Permit p = new Permit(this.sem).acquire()) {
    if (x > 1000) { throw new Exception("Too large!"); }
    System.out.printf("x = %d", x);
  }
  ```
  *(also Acs.5)*

- **Ada.8 Inject dependencies only through the constructor** -> add to §2: The only legitimate "injection" is passing already-built collaborators to a constructor; do not annotate, do not use field/setter injection, and never pass the injector itself. Why: constructor injection is just ordinary Java object composition, and everything a container adds is noise. Example (Java):
  ```java
  public final class Budget {
    private final DB db;
    public Budget(final DB data) {
      this.db = data;
    }
  }
  ```

- **Acs.3 Keep `new` out of methods; push it into constructors** -> add to §2: Instantiate collaborators as far up the object lifecycle as possible — ideally in secondary constructors — and leave method bodies free of `new`. Why: the more `new` operators remain inside methods, the less reusable and testable the class is, because the dependency is an unbreakable link created at call time. Example (Java):
  ```java
  final class Story {
    private final Text text;
    Story() { this(new File("/tmp/story.txt")); }
    Story(File f) { this(new TextOf(f)); }
    Story(Text t) { this.text = t; }
    String text() { return this.text.asString(); }
  }
  ```

- **Acs.4 Avoid two-step initialization** -> add to §2: Do not add `init()`, `setup()`, `open()` or any method that must be called after the constructor before the object is usable; a constructor must be sufficient for all scenarios. Why: an `init()` method is a flag saying "I failed to design this class properly" — it introduces temporal coupling, leaves objects in an incomplete state, and masks mutability and fragility. Example (Java):
  ```java
  final class Book implements Closeable {
    private final InputStream in;
    Book(InputStream stream) { this.in = stream; }
    @Override public void close() throws IOException { this.in.close(); }
  }
  ```

- **Acs.6 Keep constructors code-free to avoid fragile base classes** -> add to §2: Constructors only assign arguments; never call overridable methods from within a constructor. Why: a base constructor calling a virtual method invokes the derived override before the derived fields are set, printing `null` and producing the "fragile base class" bug. Example (Java):
  ```java
  final class Product {
    private final String title;
    Product(String t) { this.title = t; }
    void print() { System.out.printf("Title: %s%n", this.title); }
  }
  ```

- **Apa.6 Let a prototype/proof-of-concept be built exactly one way, by one architect** -> add to §2: Build the minimal end-to-end skeleton once, then let the body grow via many small increments. Why: a one-man skeleton forces all decisions into the open, then parallel bug-fixing adds the "meat". Example:
  ```text
  Building phase: deliverables = working software; duration 2-5 days; participants: architect only.
  Fixing phase: deliverables = bug fixes via pull requests; participants: many.
  ```

- **Apa.7 Compose an object from small protocols and switch behavior by injecting a different implementation** -> add to §2: Assemble behavior at construction time by handing the object a different collaborator (e.g. a fake pass during integration tests) rather than branching inside the object. Why: constructor-injected strategy is testable and avoids `if`-based mode flags. Example:
  ```java
  new TkAuth(take, new PsChain(
      new PsFake(/* if running integration tests */),
      new PsCookie(new CcHex(new CcXOR(new CcPlain())))));
  ```

- **Apb.4 Retrieve an object, then act on it** -> add to §2: Prefer `books.findById(42)` followed by
  `b.remove()` over a collection-level `books.removeById(42)`, so the retrieved object carries its own
  identity and can be decorated/lifecycle-managed individually. Why: it keeps retrieval and mutation
  responsibilities separated and lets a small, focused object (the book) own its own behavior rather
  than the collection doing everything. Example:
  ```text
  b = books.findById(42)
  b = Logged.new(b)
  b.remove        # decorator logs deletion; collection never knows
  ```

### Additions from real codebases

- **Qulice enforces code-free constructors mechanically.** `ConstructorsCodeFreeCheck` permits only
  field assignment and `this(...)`/`super(...)`, with a narrow exception for defensive
  `Arrays.copyOf`/`.clone()` and calls inside lambdas/anon classes (`checks.xml:486`); the PMD trio
  `ConstructorShouldDoInitialization`, `OnlyOneConstructorShouldDoInitialization`,
  `ConstructorOnlyInitializesOrCallOtherConstructors` (`ruleset.xml:688-739`) plus
  `ImplicitConstructorCheck` (force an explicit ctor for Javadoc, `checks.xml:479`) and
  `ConstructorsOrderCheck` (`checks.xml:485`).
- **The rule has teeth.** Takes' revapi allow-list records Qulice *forcing removal* of ctors that
  called `String.split`, `Pattern.compile`, `String.getBytes`, `URI.create`, `URL.openStream`,
  `SSLServerSocketFactory...`, `String.format`, and signature changes `String`→`byte[]` and
  `String`→`CharSequence` (`pom.xml:468-641`, e.g. `PsToken(String)`→`(byte[])` at `:589-598`).
- **The positive pattern: ctors assign or build a composed delegate.** Cactoos `Sticky` builds the
  inner `StickyFunc` in the ctor and runs logic in `value()` (`scalar/Sticky.java:53-62`);
  `Contains` chains four overloads to one primary (`text/Contains.java:33-66`);
  `ScalarEnvelope`/`TextEnvelope` hold one `private final` origin and delegate. requs `RegexRule` is
  two assignments only (`RegexRule.java:43-46`).
- **`new` is pushed to the edge, not into methods.** Takes' `TkGzip` ctor composes an `RsFork` lambda
  and passes it to `super(...)` (`tk/TkGzip.java:89-100`); `BkParallel` delegates a composed `Back`
  (`http/BkParallel.java:84-105`); the README's rule is "no `new` inside methods".
- **Build reproducibility belongs with ctor discipline.** eo pins `maven-compiler-plugin` to 3.8.1 and
  even excludes it from Renovate (`pom.xml:395-396`, `renovate.json:6-17`), and commits
  `.mvn/jvm.config` (`-Xmx4096m -Xms1024m -XX:+HeapDumpOnOutOfMemoryError`) for a reproducible JVM.
- **Two-step initialization appears only for JDK/third-party seams.** rehttp declares an empty
  `<argLine/>` property and composes `@{argLine} -Djava.awt.headless=true` so coverage agents can
  inject at runtime (`rehttp/pom.xml:60,206-217`) — a build-level "assign first, act later".

## 3. Immutability and state

- **B3.1 Make all fields `private final`** — Immutability starts with `final` on every property.
  Why: `final` instructs the compiler that modification outside the ctor is an error, guaranteeing
  the object never changes after creation.
  Example (Java):
  ```java
  final class Cash {
      private final int dollars;                // immutable
      Cash(int val) { this.dollars = val; }
  }
  ```

- **B3.2 Return a new object instead of mutating** — Modifying operations must build and return a new
  object. Why: `five` must always mean five; mutating `five` into fifty via `five.mul(10)` makes code
  confusing and unmaintainable.
  Example (Java):
  ```java
  final class Cash {
      private final int dollars;
      Cash(int val) { this.dollars = val; }
      Cash mul(int factor) { return new Cash(this.dollars * factor); }  // new object
  }
  Cash five = new Cash(5);
  Cash fifty = five.mul(10);
  ```

- **B3.3 Immutability kills the "identity mutability" bug** — A key's state must never change after
  it is placed in a hash-based collection. Why: if you mutate a key (e.g. `five.mul(2)`), the `Map`
  keeps stale hash entries and can end up with two "equal" keys and unpredictable lookups.
  Example (Java):
  ```java
  Map<Cash, String> map = new HashMap<>();
  Cash five = new Cash(5); Cash ten = new Cash(10);
  map.put(five, "five"); map.put(ten, "ten");
  // With immutable Cash you cannot corrupt the map; with mutable Cash:
  // five.mul(2);  // map becomes {$10=>"five", $10=>"ten"}
  ```

- **B3.4 Immutability gives failure atomicity** — An operation either produces a complete new object
  or fails; it never half-modifies state. Why: with mutable objects, an exception between two field
  assignments leaves a "broken" object (dollars updated, cents not), which is very hard to debug.
  Example (Java):
  ```java
  // mutable + manual rollback (messy, error-prone)
  void mul(int factor) { int before = this.dollars; this.dollars *= factor; if (/* bad */) { this.dollars = before; throw new IllegalStateException("oops"); } this.cents *= factor; }
  // immutable: atomic by construction
  Cash mul(int factor) { if (/* bad */) { throw new IllegalStateException("oops"); } return new Cash(this.dollars * factor, this.cents * factor); }
  ```

- **B3.5 Immutability removes temporal coupling** — Instantiation and initialization must be one
  statement, so the order of statements can never matter. Why: with setters, correctness depends on
  calling `setDollars` before `setCents` before `println`; forgetting or reordering lines still
  compiles but breaks logic.
  Example (Java):
  ```java
  // wrong: order-sensitive
  Cash price = new Cash();
  price.setDollars(29);                       // 50 lines later...
  price.setCents(95);                         // 30 lines later...
  System.out.println(price);
  // right: order cannot be broken
  Cash price = new Cash(29, 95);
  System.out.println(price);
  ```

- **B3.6 Immutability eliminates side effects** — Nobody can modify an immutable object passed to
  them; a callee cannot secretly change your `five` into `ten`. Why: side effects force you to debug
  every place an object was touched; immutability guarantees the object always means what it says.
  Example (Java):
  ```java
  // with immutable Cash, this callee cannot change five:
  void print(Cash price) { System.out.println("Today: " + price); }
  ```

- **B3.7 Immutability forbids unset NULL properties** — No field may start as `null` and be filled in
  later. Why: mutable "temporarily unset" fields let one big class masquerade as user/customer/SQL
  record depending on initialization state; immutability forces small, solid, cohesive classes.
  Example (Java):
  ```java
  // wrong
  class User { private final int id; private String name = null; User(int n) { this.id = n; } void setName(String t) { this.name = t; } }
  // right: split roles into separate immutable classes
  final class Customer { private final String name; Customer(String n) { this.name = n; } }
  ```

- **B3.8 Immutability gives thread safety for free** — Immutable objects are safe to share across
  threads without synchronization. Why: mutating `dollars` and `cents` in two threads can produce a
  torn state (e.g. `$60.20`); explicit `synchronized` costs performance and risks deadlocks.
  Example (Java):
  ```java
  final class Cash { private final int dollars; private final int cents; Cash(int d, int c) { this.dollars = d; this.cents = c; } }
  // No synchronization needed; state can never change.
  ```

- **B3.9 Immutability produces smaller, simpler objects** — Immutable classes naturally resist
  growth because the ctor would get visibly ugly, prompting decomposition. Why: simplicity is the
  most important virtue of modern programming; shorter classes are easier to understand and refactor.
  Example (Java):
  ```java
  // Growing immutable Cash pushes you to extract sub-objects instead of a 10-arg ctor:
  final class Cash { private final Digits digits; private final Currency currency; /* ... */ }
  ```

- **B3.10 Laziness needs a controlled workaround, not mutability** — Lazy loading in Java requires a
  mutable cache field or a framework/static map; prefer an explicit caching decorator, or a language
  feature like `@OnlyOnce`. Why: lazy loading is about performance and must not justify mutable
  classes; the language should provide the feature rather than the object becoming mutable.
  Example (Java):
  ```java
  // anti-pattern: mutable lazy field with null sentinel
  class Page { private final String uri; private String html = null; String content() { if (this.html == null) { this.html = load(); } return this.html; } }
  // better: wrap with a caching decorator (see B2.8)
  ```

- **B3.11 Distinguish "immutable" from "constant"** — An immutable object never changes its state;
  a constant object's state *is* the real entity it represents. `WebPage` is immutable even though
  `content()` returns different bytes each call; `String` is a constant. Why: confusing the two makes
  people think mutable entities (web pages, memory) require mutable objects — they do not.
  Example (Java):
  ```java
  class WebPage {                              // immutable, but not constant
      private final URI uri;
      WebPage(URI path) { this.uri = path; }
      String content() { /* HTTP GET; value varies */ return ""; }
      void modify(String content) { /* HTTP PUT; object state unchanged */ }
  }
  ```

- **B3.12 State = coordinates of the represented real entity** — An object's state is the set of
  coordinates used to find its real-world entity (URI, file path, memory offsets). Why: once state is
  understood as coordinates, it is clear that mutating coordinates means switching to a different
  entity — an act of disloyalty.
  Example (Java):
  ```java
  class File { private final java.nio.file.Path path; File(java.nio.file.Path p) { this.path = p; } }
  // path is a coordinate; the object stays loyal to that one file forever.
  ```

- **B3.13 An immutable object's identity equals its state; override `equals`/`hashCode`** — Two
  immutable objects with equal state are identical; implement `equals()` and `hashCode()` from the
  encapsulated state. Why: in a perfect OOP world identity is not separate from state; Java's default
  shell-identity is a flaw, so EO compensates by overriding equality.
  Example (Java):
  ```java
  class WebPage {
      private final URI uri;
      WebPage(URI path) { this.uri = path; }
      @Override public boolean equals(Object obj) { return obj instanceof WebPage && this.uri.equals(((WebPage) obj).uri); }
      @Override public int hashCode() { return this.uri.hashCode(); }
  }
  ```

- **B3.14 Model real mutable resources with immutable objects** — Memory, disk, the network are all
  external resources; an immutable object merely holds their coordinates and issues operations.
  Why: a C++ `ImmutableList` with `int* const` fields is conceptually a `WebPage` with a URI — the
  pointer is just another coordinate, so mutable *state* never follows from mutable *resources*.
  Example (C++):
  ```cpp
  class ImmutableList {
  public:
      ImmutableList(): total((int*) calloc(1, sizeof(int))), items((int*) malloc(100)) {}
      void add(int number) { int pos = *total; items[pos] = number; *total = pos + 1; }
  private:
      int* const total;   // constant pointers: state never changes
      int* const items;
  };
  ```

- **B3.15 Mutable objects have no right to exist** — Do not "choose" mutability for games, UI,
  mobile, web or algorithms; all domains can and must be modeled with immutable objects. Why: there
  is no domain where mutability is required once you treat external resources as coordinates; mutable
  objects are simply an abuse of the paradigm.
  Example (Java):
  ```java
  // Always: mutate by replacement, never by assignment.
  Cash updated = cash.mul(2);
  ```


### Additions from articles

- **Acoa.10 Separate "state" from "behavior" so frequently changed data does not force mutation** ->
  add to §3: A document's `title` is not state if it changes often; keep `id` as state and expose
  `title()`/`title(String)` as *behavior* that reads/writes the real storage. Why: an immutable
  proxy object may have methods that return different values each call without ever changing its
  encapsulated coordinates. Example (Java):
  ```java
  @Immutable
  interface Document {
    String title();          // behavior: read from storage
    void title(String text); // behavior: write to storage
  }
  ```

- **Acoa.12 Use immutability to force small, cohesive classes** -> add to §3: Because an immutable
  class takes all state through (a small) constructor, you are forced to break up a growing class
  into stamps/decorators instead of adding setters. Why: `commons-email`'s `Email` grew to 33
  fields / 100+ methods precisely because setters let it absorb every new feature; `jcabi-email`
  split it into `Postman`, `Envelope`, `Stamp` objects. Example (Java):
  ```java
  @Immutable
  interface Stamp { void attach(Message message); }
  // Envelope.MIME holds Array<Stamp> and stays 25 lines.
  ```

- **Acoa.13 Know the concrete costs of mutability you avoid** -> add to §3: Immutability buys
  thread-safety, temporal-coupling avoidance, side-effect-free sharing, and failure atomicity;
  mutable `Date` demonstrates the identity-mutability trap (after `setTime`, `map.containsKey`
  returns false because the hash changed). Example (Java):
  ```java
  Map<Date, String> map = new HashMap<>();
  Date date = new Date();
  map.put(date, "hello, world!");
  date.setTime(12345L);
  assert map.containsKey(date); // false — key identity mutated
  ```

- **Acoa.14 Move caching out of the object into a decorator / aspect** -> add to §3: Do not use
  `null` fields + `synchronized` for lazy loading; either wrap with `CachedName`/`@Cacheable`, or
  solve it at another layer. Why: an object responsible for its own caching becomes mutable and
  takes on platform performance concerns (`Department.manager()` example). Example (Java):
  ```java
  public final class CachedName implements Name {
    private final Name origin;
    public CachedName(final Name name) { this.origin = name; }
    @Cacheable(forever = true)
    public String first() { return this.origin.first(); }
  }
  ```
  *(also Adt.11)*

- **Acoa.15 If you must mutate in-memory data, use a `final` byte array or a Memory-like holder** ->
  add to §3: Java lacks a `Memory` class; a `final` array is the only in-memory structure mutable
  through a `final` reference, so it is the pragmatic surrogate. Why: an in-memory title has no
  real-world file behind it, so the object's only honest "entity" is the heap bytes. Example:
  ```java
  // conceptually: private final Memory memory; title() reads/writes it.
  ```

- **Acob.14 Immutability has gradients; only "loyalty" is required** -> add to §3: keep all encapsulated attributes `private final` and expose no setters, but allow methods that dynamically compute results (time, files, in-memory mutable buffers) — the object is still immutable. Why: a constant returns the same value always; a "not a constant" may return different values (timestamps); "represented mutability" wraps a file; "encapsulated mutability" hides a `StringBuffer`. All are immutable *because they are loyal* to what they encapsulate.
  ```java
  final class Book {
    private final StringBuffer buffer;      // mutable encapsulated
    String title() { return this.buffer.toString(); }
  }
  // immutable, not a constant, not thread-safe
  ```

- **Acob.15 Immutability's defining trait is loyalty, not constancy** -> add to §3: a good immutable object never exposes a setter and never lets callers mutate the entity it represents; beyond that, behavior is flexible. Why: the author states that "loyalty to the encapsulated entities" is the *only* quality that separates mutable from immutable objects.
  ```java
  final class Book {
    private final Path path;               // faithful representative of a file
    String title() { return new String(Files.readAllBytes(this.path)); } // represents changing state
  }
  ```
  *(also Acoa.11)*

- **Acob.16 Provide synchronized decorators instead of thread-safe classes** -> add to §3: keep the core class non-thread-safe and add thread safety via a `Sync*` decorator that synchronizes delegation. Why: blocking synchronization slows every call, and mixing rich behavior with thread safety violates SRP; decorate only where concurrency is actually needed.
  ```java
  final class SyncPosition implements Position {
    private final Position origin;
    SyncPosition(Position pos) { this.origin = pos; }
    @Override public synchronized void increment() { this.origin.increment(); }
  }
  Position position = new SyncPosition(new SimplePosition());
  ```

- **Acob.17 Build transformation pipelines as immutable `with()` chains** -> add to §3: a collection object should expose `with(item)` returning a new instance rather than `add(item)` mutating itself; use a `Temporary.back()` interface to return to the base type after generic decoration. Why: `with` preserves immutability; `Train.Temporary<T>` lets `TrClasspath` satisfy "methods only from interfaces" while still exposing the wrapped train.
  ```java
  interface Train { Train with(Shift shift); Iterator<Shift> iterator(); }
  interface Train.Temporary<T> { Train<T> back(); }
  Train<Shift> train = new TrBulk<>(new TrClasspath<>(new TrDefault<>()))
      .with(names).back();
  ```

- **Acob.18 Use veil objects to precompute data without turning objects into DTOs** -> add to §3: decorate a live object with a `Veil` carrying already-fetched values; preset methods return the cached data, any other call "pierces" the veil and delegates to the real object. Why: mapping DB rows to `Project` objects causes N+1 round-trips; a veil keeps the object OO and efficient, while `Unpiercable` is for read-only precomputation.
  ```ruby
  @pgsql.exec('SELECT * FROM project').map { |r|
    Veil.new(Project.new(@pgsql, r['id'].to_i), name: r['name'], author: r['author'])
  }
  ```

- **Atq.4 Make thread-safety explicit and tested, don't trust a thread-safe field** -> add to §3: A class holding a `ConcurrentHashMap` is not automatically thread-safe; synchronize the compound operation and prove it with a parallel test. Why: `map.size() + 1` followed by `put` is a check-then-act race invisible to single-thread tests. Example (Java):
  ```java
  final class Books {
    private final Map<Integer, String> map = new ConcurrentHashMap<>();
    synchronized int add(String title) {
      final int next = this.map.size() + 1;
      this.map.put(next, title);
      return next;
    }
  }
  ```

- **Adt.9 Annotate every public interface `@Immutable`** -> add to §3: Mark interfaces (e.g. `Request`, `Response`) with `@Immutable` so instances can be safely encapsulated inside other immutable objects. Why: callers must be able to trust that a passed-in object never mutates; e.g. jcabi-http requires it. Example:
  ```java
  @Immutable
  public interface Request { /* ... */ }
  ```

- **Adt.10 Hide/move mutable scheduling out of fields; pass dependencies in** -> add to §3: Initialize persistence/connection objects in `Entry` and pass them as constructor arguments down into `TkApp`/takes; never mutate a field after construction. Why: container-free, constructor-injected dependencies keep every object immutable and testable. Example:
  ```java
  public static void main(final String... args) throws Exception {
    new FtCli(new TkApp(Entry.postgres()), args).start(Exit.NEVER);
  }
  ```

- **Ada.11 Implement lazy loading with a `Scalar`, not a null field** -> add to §3: Represent a not-yet-loaded attribute as a function (`Scalar`/lambda) held in a `final` field; execute it in the method that needs it. Why: the classic lazy-load pattern mutates a field and sets it to `null`, violating immutability and the no-null rule; a function-in-a-field keeps the object immutable. Example (Java):
  ```java
  final class Encrypted4 implements Encrypted {
    private final IoCheckedScalar<String> text;
    Encrypted4(final Scalar<String> source) {
      this.text = new IoCheckedScalar<>(source);
    }
    public String asString() throws IOException {
      final byte[] in = this.text.value().getBytes();
      // ...
    }
  }
  ```
  *(also Ada.13)*

- **Ada.12 Cache with `StickyScalar`, synchronize with `SyncScalar`** -> add to §3: When a lazy scalar must be evaluated only once, wrap it in a sticky decorator; when it may be shared across threads, wrap the sticky one in a synchronized decorator. Why: a plain lazy function re-reads an exhausted stream on every call, and a cached mutable value is not thread-safe, so both decorators together give immutability + caching + safety. Example (Java):
  ```java
  Encrypted5(final Scalar<String> source) {
    this.text = new IoCheckedScalar<>(
      new SyncScalar<>(
        new StickyScalar<>(source)
      )
    );
  }
  ```

- **Acs.8 Prefer inline values + monikers over reassigned variables** -> add to §3: When the same immutable value is needed more than once, keep it as a single `final` "moniker" and inline all other values directly. Why: values used once should never be stored; values used repeatedly are constants, not variables, and naming them once keeps the code short and the number of identifiers (the biggest readability cost) low. Example (Java):
  ```java
  final Secret secret = new Secret();
  new Farewell(
    new Attempts(new VerboseDiff(new Diff(secret, new Guess())), 5),
    secret
  ).say();
  ```

- **Acs.9 Never expose state as naked data** -> add to §3: Even a `private` field with a getter/setter is naked data; hide it and offer only behavior. Why: `getT()`/`setT()` let callers decide units, precision and interpretation outside the class, re-creating tight hidden coupling under an object-like facade. Example (Java):
  ```java
  final class Temperature {
    private final int t;
    Temperature(int value) { this.t = value; }
    public String toString() { return String.format("%d F", this.t); }
  }
  ```

- **Apa.8 Make immutability a project-wide invariant, not a local choice** -> add to §3: Forbid mutable objects, public properties and global variables in the coding standard and encode it in static analysis. Why: mutable shared state is the root of most chaos; immutability must be enforced mechanically. Example:
  ```text
  Coding standard: all fields private final; no setters; no public fields;
  no global variables; no mutability of method arguments.
  ```
  *(also Acs.7)*

- **Apa.9 Treat history as immutable: never rewrite, delete or force-push** -> add to §3: Do not force-push, do not delete commits, do not delete ticket comments; let the messy history stand. Why: traceability (what/who/why per change) is maintainability; destroying history destroys the project's memory. Example:
  ```text
  # FORBIDDEN: git push --force          (overwrites remote history)
  # FORBIDDEN: deleting GitHub comments  (destroys reasoning trail)
  ```

- **Apb.5 Make objects immutable during refactoring** -> add to §3: When taking over foreign code,
  convert mutable fields/classes to immutable ones as a dedicated refactoring step; measure with
  jpeek (most projects have ~80% mutable classes). Why: immutability keeps objects smaller and is
  "purely profitable" — no behavior change, lower state surface, fewer bugs. Example:
  ```text
  // step order: remove red spots -> remove empty lines -> short names -> add tests
  //   -> single return -> remove NULLs -> make immutable -> remove static -> Qulice
  ```

- **Apb.6 Immutability is a ranking goal, track null/static ratios** -> add to §3: Track the density of
  `null` and `static` keywords per LoC as objective quality indicators, and drive them down. Why:
  Takes had ~1 null per 2,700 lines vs Spring Boot's ~1 per 35; static was ~1 per 496 lines vs ~1 per
  32 — concrete numbers make "clean code" measurable. Example:
  ```text
  # audit: count `null`, `static`, mutable classes, long names, empty lines
  git grep -c '\bnull\b' -- '*.java'
  ```

### Additions from real codebases

- **Every field is `private final` and every class is `final`/`abstract` in the strict repos.**
  Cactoos: 295 `public final class` / 312 `private final` field declarations; `Sticky`, `Synced`
  (thread-safety) and `NoNulls` (validation) are separate one-responsibility wrappers; all extend an
  `XxxEnvelope` base. Takes: 297 `final class`, 1 `abstract class`, 32 interfaces.
- **Qulice's immutability cluster.** `FinalParameters`, `FinalLocalVariable`, `FinalClass`,
  `VisibilityModifier` (private fields), `ParameterAssignment` forbidden, `ArrayIsStoredDirectly`,
  `ProhibitNonFinalClassesCheck`, `SingleUseConstantCheck` (`checks.xml:209-264,220,493-498`).
- **`@Immutable` as a contract.** s3auth marks interfaces/classes with jcabi-aspects `@Immutable`
  (`Host.java:18`, `Bucket.java:16`) and decorates behavior: `FastHost` (`@Timeable`), `SmartHost`,
  `RejectingHost`, `GzipResource`, `SyslogResource`, each holding `private final transient Host origin`.
  `default_strength` aside, this mirrors book `Adt.9`.
- **Immutable exposure at the boundary.** s3auth `DomCursor` wraps
  `Collections.unmodifiableCollection(nds)` (`DomCursor.java:38`); `RejectingHost` stores
  `jcabi-immutable Array<String>`; `GzipResource` uses Guava `ImmutableList`. rehttp/requs use
  `Collections.unmodifiableCollection(this.dirs)` (`XeFacet.java:107`).
- **Thread safety is a decorator, not a field.** Cactoos `Synced`; Qulice deliberately *excludes*
  PMD `AvoidInstantiatingObjectsInLoops` because constructing a fresh immutable object per iteration
  is the EO way (`ruleset.xml:539-545`), and excludes `UseConcurrentHashMap`/`DoNotUseThreads`
  (`ruleset.xml:554-567`).
- **Honest deviations observed.** xembly's `Directives` is a documented mutable, thread-safe builder
  (Javadoc says so); `MkHost.stats()` returns `null`; `DefaultHost` null-checks a `Resource` field
  (`DefaultHost.java:135-153`); rehttp's Heroku `system.properties` pins `java.runtime.version=1.8`
  while CI builds on 21.

## 4. Names, interfaces and contracts

- **B4.1 Name a class by what it IS, not by what it does** — Derive the name from the entity the
  objects encapsulate. Why: capability/functionality-based names describe an activity, not an entity;
  `PrimeNumbers` (a list of primes) beats `PrimeFinder`/`PrimeChooser`/`PrimeHelper`.
  Example (Java):
  ```java
  // wrong
  class PrimeFinder { java.util.List<Integer> find(java.util.List<Integer> src) { return null; } }
  // right
  class PrimeNumbers implements Iterable<Integer> { private final Iterable<Integer> origin; /* ... */ }
  ```

- **B4.2 Never use an "-er" (or "-or") suffix in a class name** — Ban Manager, Controller, Helper,
  Handler, Writer, Reader, Converter, Validator, Router, Dispatcher, Observer, Listener, Sorter,
  Encoder, Decoder. Why: the suffix signals procedural procedure-bundling, not an entity. Accept the
  rare fossilized nouns (`computer`, `user`).
  Example (Java):
  ```java
  // wrong: Target? EncodedText? DecodedData? SortedLines? ValidPage? -> use these
  class EncodedText { private final String value; EncodedText(String v) { this.value = v; } String text() { return this.value; } }
  class SortedLines implements Iterable<String> { /* ... */ }
  class ValidPage { private final boolean valid; ValidPage(boolean v) { this.valid = v; } }
  ```

- **B4.3 Builders are nouns, manipulators are verbs** — A method that returns a value is a builder
  and has a noun name; a method that modifies the world returns `void` and has a verb name. Why:
  this mirrors telling an object what result you want vs. asking it to perform an action; never mix
  the two.
  Example (Java):
  ```java
  // builders (nouns, return value)
  int sum(int x, int y);
  float speed();
  String parsedCell(int x, int y);
  // manipulators (verbs, void)
  void save(String content);
  void put(String key, Float value);
  void quicklyPrint(int id);
  ```

- **B4.4 A method is either a builder or a manipulator — never both** — If it returns a value it
  must not modify; if it modifies it must return `void`. Why: a method that saves *and* counts bytes
  is unfocused and has no clean name; split the concept into a new class.
  Example (Java):
  ```java
  // wrong: writes and reports bytes at once
  int write(InputStream content);
  // right: a builder returns a pipe; the pipe does the writing
  OutputPipe output();
  class OutputPipe { void write(InputStream content) { /* ... */ } int bytes() { return 0; } long time() { return 0L; } }
  ```

- **B4.5 Boolean builders are adjectives, without an `is` prefix** — A boolean-returning method reads
  best as an adjective; mentally prepend `is` to check it sounds right, then omit it. Why: `if
  (name.empty())` reads as "if name is empty"; `get`/verb names fail this test.
  Example (Java):
  ```java
  boolean empty();     // "is empty"
  boolean readable();  // "is readable"
  boolean negative();  // "is negative"
  // wrong: boolean equals(Object obj); boolean exists();
  // right: boolean equalTo(Object obj); boolean present();
  ```

- **B4.6 Prefer method overloading to distinct method names** — Declare same-named methods with
  different arguments where the language allows. Why: overloads read like business language and keep
  related contracts together; overloading is a fundamental OOP feature (absent in Ruby/PHP).
  Example (Java):
  ```java
  // right
  String content(File file) { /* ... */ return ""; }
  String content(File file, Charset charset) { /* ... */ return ""; }
  // wrong
  String contentInCharset(File file, Charset charset) { /* ... */ return ""; }
  ```

- **B4.7 Always use interfaces to decouple objects** — Every service must be exposed through an
  interface (contract); dependents should hold the interface, not the implementation. Why: interfaces
  allow one class to be modified or replaced without touching its clients, enforcing loose coupling.
  Example (Java):
  ```java
  interface Cash { Cash multiply(float factor); }
  final class DefaultCash implements Cash {
      private final int dollars;
      DefaultCash(int dlr) { this.dollars = dlr; }
      @Override public Cash multiply(float factor) { return new DefaultCash(this.dollars * (int) factor); }
  }
  final class Employee { private final Cash salary; Employee(Cash c) { this.salary = c; } }
  ```

- **B4.8 A public method without a contract is forbidden** — Every public method must override an
  interface method. Why: an interface-less public method lets callers couple to the concrete class
  and prevents replacing the implementation later; the service must be documented as a contract.
  Example (Java):
  ```java
  // wrong
  class Cash { public int cents() { return 0; } }         // no interface, not overridden
  // right
  interface Cash { int cents(); }
  final class DefaultCash implements Cash { @Override public int cents() { return 0; } }
  ```

- **B4.9 Keep interfaces short** — Interfaces are more expensive to grow than classes because a class
  may implement several; demand as little as possible. Why: a long interface is a demanding,
  non-cohesive contract (SRP violation) that forces implementers to build unrelated functionality.
  Example (Java):
  ```java
  // wrong: too demanding
  interface Exchange { float rate(String target); float rate(String source, String target); }
  // right: one method; convenience lives in a nested Smart class
  interface Exchange { float rate(String source, String target); }
  ```

- **B4.10 Ship a nested "smart" class with an interface for convenience methods** — Add common
  convenience/aggregation behavior as a nested class (e.g. `Exchange.Smart`) that delegates to the
  short interface. Why: shared functionality is written once and every implementation stays lean;
  extend the interface's usability without growing the contract.
  Example (Java):
  ```java
  interface Exchange {
      float rate(String source, String target);
      final class Smart {
          private final Exchange origin;
          Smart(Exchange exch) { this.origin = exch; }
          float toUsd(String source) { return this.origin.rate(source, "USD"); }
          float eurToUsd() { return this.toUsd("EUR"); }
      }
  }
  float rate = new Exchange.Smart(new NYSE()).eurToUsd();
  ```

- **B4.11 Never name a method with a `get`/`set` prefix** — `getDollars()` says "dig into your data",
  while `dollars()` asks "how many dollars do you have?". Why: the prefixes advertise the object as a
  naked data structure; the evil is the prefix, not the idea of returning data.
  Example (Java):
  ```java
  // wrong
  class Cash { private final int value; public int getDollars() { return this.value; } }
  // right
  class Cash { private final int value; public int dollars() { return this.value; } }
  ```

- **B4.12 Replace public constants and enums with micro-classes** — Every shared literal (e.g.
  `CRLF`, `"POST"`) becomes a class that encapsulates its semantics. Why: more small, non-duplicating
  classes makes code more readable (like more words in prose); a public constant is dumb, global and
  coupling.
  Example (Java):
  ```java
  // wrong
  new HttpRequest().method(HttpMethods.POST).fetch();
  // right
  new PostRequest(new HttpRequest()).fetch();
  ```

- **B4.13 A class must not be a data structure with accessors decoratged as methods** — Even with
  validation inside, a getter/setter pair is a data-access seam. Why: on the surface it is an entry
  point to bytes; the object looks like a struct no matter what the body does. Replace with behavior
  that *tells*.
  Example (Java):
  ```java
  // wrong
  class Cash { private int dollars; public void setDollars(int v) { this.dollars = v; } }
  // right
  class Cash { private final int dollars; Cash(int d) { this.dollars = d; } Cash add(Cash extra) { return new Cash(this.dollars + extra.dollars); } }
  ```

- **B4.14 Let clean names replace documentation** — Make the code self-explanatory with precise
  class/method names; do not write Javadoc for internals. Why: bad design forces documentation; a
  well-named design (`department.employee("Jeff")`, `jeff.giveRaise(...)`) needs none. Document only
  external interfaces.
  Example (Java):
  ```java
  Employee jeff = department.employee("Jeff");
  jeff.giveRaise(new Cash("$5,000"));
  if (jeff.performance() < 3.5) { jeff.fire(); }
  ```

- **B4.15 Method name should reveal the object's mission and purpose** — A proper name tells the user
  what the object is for; an improper one encourages treating it as a bag of procedures. Why: names
  shape how objects are used — respectful naming produces respectful usage.
  Example (Java):
  ```java
  // wrong: procedural request
  InputStream load(URL url); String read(File file);
  // right: object produces the result
  InputStream stream(URL url); String content(File file);
  ```


### Additions from articles

- **Acoa.16 Name the object after what it *is*; use `Sorted` not `Sorter`** -> add to §4: An `-er`
  name turns a partner object into a dumb imperative executor; `Sorted` can decide *how* to answer
  `get(0)` (scan for the largest) instead of being forced to sort first. Why: the `-er` suffix is a
  "sign of disrespect" and hides the real intention (finding the biggest apple). Example (Java):
  ```java
  List<Apple> sorted = new Sorted(apples);   // good: object behaves like a sorted list
  List<Apple> sorted = new Sorter().sort(apples); // bad: procedure pretending to be a class
  ```

- **Acoa.18 Drop `get`/`set` prefixes — ask the object to tell, not to hand over data** -> add to
  §4: `dog.give()` / `dog.weight()` replaces `dog.getBall()` / `dog.setWeight()`; the object decides
  what happens after the request. Why: `getX` signals we treat the object as a data holder; the
  "tell me your name" framing is object thinking. Example (Java):
  ```java
  Dog dog = new Dog("23kg");
  int weight = dog.weight(); // not dog.getWeight()
  ```

- **Acoa.19 Rename `FileReader` to `DataFile` (or `FileWithData`)** -> add to §4: A file that has
  content is a more powerful *file*, not a "reader"; the name should say what it is. Why: you can
  draw "a file with data", you cannot draw "a reader". No snippet needed.

- **Acoa.20 Prefer printers/media over getters** -> add to §4: Instead of exposing fields, give the
  object `toXML()`/`print(Media)` so no data escapes; use a `Media` object (`JsonMedia.with(...)`)
  when many formats are needed, keeping the class small. Why: getters turn a self-sufficient object
  into "a bag of data". Example (Java):
  ```java
  public Media print(Media media) {
    return media.with("isbn", this.isbn).with("title", this.title);
  }
  ```
  *(also Ada.14, Ada.15)*

- **Acob.19 Name interfaces as nouns and prefix implementing classes** -> add to §4: every interface is a noun (`Request`, `Directive`, `Domain`); every class implements one and is prefixed (`RqBuffered`, `RsWithType`, `DyDomain`). Why: prefixes tell the reader which type a class *is*; the author's medium projects use only 5–15 prefixes total, keeping names short (longest in Takes is `RqWithDefaultHeader`).
  ```java
  // interface Request; classes: RqBuffered, RqSimple, RqLive, RqWithHeader
  ```

- **Acob.20 Ban the `Client` suffix; model the server's entities** -> add to §4: a `*Client` represents an entire server (too broad, data-focused, hard to test/decorate); instead expose small client-side objects named after server entities (`Bucket`, `Object`, `Version`, `Policy`). Why: `AmazonS3Client` grew to 160+ methods returning DTOs, forcing duplication and a God class; a small `Region` object anchors a graph of entity objects.
  ```java
  interface Region { Bucket bucket(String name); }
  interface Bucket { Object object(String key); void delete(); }
  ```

- **Acob.22 Avoid giant interfaces; fluent APIs force classes to bloat** -> add to §4: reject interfaces with dozens of methods (e.g. `Stream`'s 43) and fluent chains that only grow by adding methods; use small composable decorators. Why: every new JDK/API feature forces a new method on `Request`/`Response`; `.as(RestResponse.class)` exists only because `Response` can't hold 50 methods, and it needs reflection/casting.
  ```java
  // instead of fluent: request.method("GET").fetch().assertStatus(200).body()
  String html = new BodyOfResponse(
      new ResponseAssertStatus(
          new RequestWithMethod(new JdkRequest(url), "GET"), 200)).toString();
  ```

- **Acob.23 Hide parsing inside a parsing object** -> add to §4: deserialize JSON/XML by a class that `implements Book` and computes fields lazily from the raw text, not by a DTO with setters and a procedural HTTP handler. Why: the DTO approach is not reusable, repeats validation, and has temporal coupling; `new JsonBook(body)` moves parsing into the object.
  ```java
  final class JsonBook implements Book {
    private final String json;
    JsonBook(String body) { this.json = body; }
    @Override public String isbn() {
      return Json.createReader(new InputStreamOf(this.json)).readObject().getString("isbn");
    }
  }
  ```

- **Acob.24 For final JDK types, wrap in a Scalar of the interface** -> add to §4: when you cannot `implements` (e.g. `String` is final), implement `Scalar<String>` and expose `value()`. Why: parsing objects sometimes must stand in for a primitive; the author used `RqUser implements Scalar<String>` in jare.
  ```java
  final class RqUser implements Scalar<String> {
    @Override public String value() { /* parse from request */ }
  }
  ```

- **Acob.25 Welcome polymorphism; don't demand concrete collaborators** -> add to §4: type collaborators by interface, never by the concrete class of "Bobby's" implementation; polymorphism is what makes the app testable and robust. Why: the author calls the wish to pin down `EmployeeHourlyRate` instead of `Money` a "fear of decoupling" (Fail Safe) that hides bugs and destabilizes the product.
  ```java
  void send(Money m) { /* ok */ }
  // avoided: void send(EmployeeHourlyRate m)
  ```

- **Atq.5 Name test classes after the one live class they validate** -> add to §4: `FooTest.java` must test `Foo.java`; `FooITCase.java` tests integration of `Foo`; anything not a test goes into a `support/` or `it/` package that has no counterpart in `src/main/java`. Why: a failing test's class name is the reader's only pointer to the broken source file. Example: layout `src/test/java/foo/{PhrasesTest.java, GreetingsTest.java, it/SimpleGuessingITCase.java, support/FooUtils.java}`.

- **Atq.6 Give every test method a behavior-describing name** -> add to §4: Name tests after the behavior they assert (`countsSimpleGreetings`), never `test1`/`testCheck`. Why: the test name is the first thing a developer reads when the build turns red. Example (Java):
  ```java
  @Test
  void countsSimpleGreetings() {
    assertThat(
      "Total count of greetings",
      new Phrases("Hello, world!").greetings().count(), equalTo(1)
    );
  }
  ```
  *(also Adt.15)*

- **Adt.12 Never name classes with `-er` or with type suffixes** -> add to §4: Rename validators/controllers/managers to real entities; do not prefix interfaces (`IRecord`) or suffix them (`RecordInterface`) — name the entity for the interface and describe implementation detail in the class. Why: `-er` names a job title, not an object; interface/class prefixes are noise. Example:
  ```java
  class SimpleUser implements User {}
  class DefaultRecord implements Record {}
  class Validated implements Content {}
  ```

- **Adt.13 Return-type name for builders, action verb for manipulators** -> add to §4: A returning method's name states what it returns (`content()`, `ageOf(File)`, `isValid(String)`); a `void` method's name states what it does (`save(File)`, `append(...)`). Why: names become self-documenting and the query/command distinction is obvious without reading the body.

- **Adt.14 Name a returning method as a noun, not `find`/`get`** -> add to §4: Prefer `book(String)` over `find(String)` or `getBook()` so the object decides whether it finds, constructs, or caches. Why: `find` leaks *how* the result is produced into the caller's vocabulary; the object should decide.

- **Ada.16 Prefer skinny interfaces that return raw data** -> add to §4: Design fewer interfaces that yield plain data which adapters/decorators "dress up", rather than a deep fat hierarchy of interfaces each returning already-functional objects. Why: a skinny design is more extendable, cohesive, reusable and testable; mocking one interface is far easier than mocking an entire type hierarchy. Example (Java):
  ```java
  interface Article {
    String head();
  }
  final class TxtHead {
    private final Article article;
    String author() { /* parse */ }
    String title()  { /* parse */ }
  }
  // vs. fat: Article.head().author().name()
  ```

- **Ada.17 Depend on the narrowest interface (interface segregation / loose coupling)** -> add to §4: Declare the parameter or field with the least capable interface that serves the need (`Iterable` over `Collection` over `List`; `List` over `ArrayList`). Why: it lets the provider decide the concrete implementation and is nothing more than the old loose-coupling rule. Example (Java):
  ```java
  // Only iterate? Ask for Iterable, not List.
  void show(final Iterable<String> items) { /* ... */ }
  ```

- **Acs.10 Name with a single noun, not a compound** -> add to §4: Give variables, parameters, fields and methods unique single-word noun names; avoid `csvFileName`, `textLength`, `current-user-email`. Why: a compound name signals a scope so large and ambiguous that a plain noun is not enough — a big scope is itself the smell; ideal methods hold ≤5 variables and ideal classes ≤5 properties. Example (Java):
  ```java
  final class Csv {
    private final File file;
    Csv(File src) { this.file = src; }
    Iterable<String> records() { /* ... */ }
  }
  ```

- **Acs.11 Use short entity prefixes to locate classes in a hierarchy** -> add to §4: For "entity with specifier" classes, prefix the shared entity instead of suffixing long adjectives: `PtRandom`, `PtOpened`, `PtTcp` rather than `RandomPort`, `OpenedPort`, `TcpPort`. Why: a small set of two-letter prefixes tells a reader instantly which interface the class implements (`Rq*` = Takes `Request`), removes repeated noun noise, and makes references shorter. Example (Java):
  ```java
  final class RqFake implements Request { /* ... */ }
  final class RsWithStatus implements Response { /* ... */ }
  ```

- **Acs.12 Do not overload methods** -> add to §4: Give each distinct behavior its own name or its own class; never define two methods under one name with different parameters. Why: multiple implementations under one name hide semantics (`add(int)` could search a catalog, re-add, etc.), forcing the caller to guess and consult a Javadoc that may be stale, which breeds bugs. Example (Java):
  ```java
  final class ProductInCatalog implements Product {
    private final Product p;
    ProductInCatalog(int id) { this.p = new Catalog().findById(id); }
  }
  cart.add(new Product("book"));
  cart.add(new ProductInCatalog(42));
  ```

- **Acs.13 Let names carry the type; drop redundant annotations** -> add to §4: Name variables after the nouns they represent (`book`, `city`) and use `var`; the name already conveys the type, so a type annotation on a local is syntactic redundancy. Why: type annotations lengthen code and lower readability; a good noun is sufficient to disambiguate, and inference keeps lines shorter. Example (Java):
  ```java
  Price priceOfDelivery(Book book, City city) {
    var delivery = new Delivery(book.price(), city);
    return delivery.price();
  }
  ```

- **Acs.14 Redesign equality around a comparable byte contract** -> add to §4: Avoid `instanceof`/casting inside `equals()`; instead expose comparison through an interface (`Digitizable.digits()`) and a separate `Comparison` object. Why: the classic `equals()` violates encapsulation (one object reads another's private fields), blocks polymorphism (cannot pass an interface implementation), and forces casts. Example (Java):
  ```java
  interface Digitizable { byte[] digits(); }
  final class Weight implements Digitizable {
    private final int kilos;
    Weight(int k) { this.kilos = k; }
    @Override public byte[] digits() {
      return ByteBuffer.allocate(4).putInt(this.kilos).array();
    }
  }
  ```

- **Apa.10 Define a glossary before anything else; one term = one meaning** -> add to §4: Every technical term in the spec must be defined in a glossary; the architect owns it. Why: "UUID vs account number" ambiguity burns hours; if the reader doesn't understand, it is the writer's fault. Example:
  ```text
  UUID is user unique ID, a positive 4-bytes integer.
  UUID is set incrementally to make sure no two users share the same UUID.
  ```

- **Apa.11 Every public behavior must sit behind an interface** -> add to §4: Introduce an interface (`DataBridge`) before an implementation (`DBWriter`); code against the abstraction. Why: interface-only public methods are mockable, decorable and testable; an interface-less class cannot be a contract. Example:
  ```java
  public interface DataBridge { void insert(String text) throws IOException; }
  public class DBWriter implements Writer { /* depends on DataBridge */ }
  ```
  *(also Acs.15, Acoa.17, Acob.21)*

- **Apa.12 Write specs as a manual/reference, never as a wish list** -> add to §4: Describe the product in present tense ("The API supports..."), never "will/must/needs to", and never leave questions, opinions or suggestions in the document. Why: spec is a contract, not a discussion board; doubt and debate belong elsewhere. Example:
  ```text
  WRONG: "The API will support JSON and XML. XML needs to be validated."
  RIGHT: "The API supports JSON and XML. XML is validated by XSD schema."
  ```

- **Apa.13 Keep functional and non-functional requirements separate** -> add to §4: State one testable functional requirement per line; put measurable quality requirements in their own section. Why: mixing "scroll the list" with "smoothly and fast" makes the requirement unverifiable, untraceable and hard to modify. Example:
  ```text
 Functional: The user scrolls the image list.
  Quality:    Any page opens in < 300 ms. Test coverage > 80%.
  ```

- **Apa.14 Give every requirement an actor and a measurable target** -> add to §4: Use user stories that start with "the user <verb>..."; make all quality requirements numeric and testable or delete them. Why: "it is possible to download" names nobody, and "UI must be attractive" cannot be verified. Example:
  ```text
  User can create account, upload photos, share photos, send messages.
  Availability must be over 99.999%; MTTR < 2 hours.
  ```

- **Apa.15 Keep the written unit of intent short and high-level** -> add to §4: Cap the Product Vision at ~2 pages, each section ≤ 20 lines, features in 2–3 lines, 4–8 PBS items. Why: if you cannot compress it you do not understand it; brevity is the strongest signal of understanding. Example:
  ```text
  Product statement: < 60 words answering: who is the customer? what do they want?
  what is the market offering now? what's wrong with it? how do we fix it?
  ```

- **Apb.7 Short names signal simple code** -> add to §4: Rename long compound identifiers such as
  `registerServletContainerInitializerToDriveServletContextInitializers` to one-word names; a name that
  cannot be shortened exposes excessive complexity hiding behind it. Why: long compound names are often
  the only documentation for poorly decomposed code and signal that many concepts are entangled in one
  method. Example:
  ```text
  registerServletContainerInitializerToDriveServletContextInitializers -> register
  ```

- **Apb.8 One exit per method** -> add to §4: Replace multiple `return` statements with a single exit
  point and decomposition into smaller methods. Why: multiple returns are acceptable in scripts
  (procedures) but in OO code they force ugly nested `if/else`, and removing them pushes you to break
  the method into smaller pieces. Example:
  ```java
  // 5 `return`s in one small method => split into small single-exit objects/methods.
  ```

- **Apb.9 Wrap identity inside the object, never leak it** -> add to §4: Return an object that
  encapsulates its own ID after lookup; never expose the raw id to callers that don't need it. Why:
  once the id is wrapped it "never has to leak out again," reducing naked-data surface and coupling.
  Example:
  ```text
  b = books.findById(42)   # id is now inside b
  b.remove                 # later, b still knows its id; caller never sees 42
  ```

### Additions from real codebases

- **Cactoos demonstrates the envelope pattern at scale.** 11 public interfaces, 10
  `@FunctionalInterface`; every public method implements an interface method
  (`Contains implements Scalar<Boolean>`); only ~10 `getX()`-shaped methods exist in the whole main
  tree. Decorators are the whole library (`README.md:378-386`).
- **Takes: short entity prefixes as class names.** `Tk*` = `Take`, `Rs*` = `Response`, `Rq*` =
  `Request`, `Fk*` = `Fork`, `Bk*` = `Back`; `Take` is `@FunctionalInterface`. The one accepted `-er`
  noun is HTTP `Header` ("a header is an entity, not a job title").
- **Qulice's naming gates are concrete regexes.** custom `MethodNameCheck`
  `^(as|at|by|go|id|in|is|it|of|on|or|to|up|[a-z]{2,}[a-zA-Z]+)$`; locals `^(id|[a-z]{3,12})$`;
  parameters `^(id|[a-z]{3,})$`; catch `^(ex|[a-z]{3,12})$`; `AbbreviationAsWordInName` allows only
  `IT`; `TypeName`/`ConstantName`/`PackageName` (`checks.xml:342-378`). `ProhibitStaticNestedClassesCheck`
  forced Takes to promote nested types to top-level (`pom.xml:360-416`). `UnusedPackagePrivateClasses`
  reports package-private types nothing references (`CheckstyleValidator.java:130-135`).
- **`Default*` implementations, `Mk*`/`*Mocker` fakes, no `get` prefixes.** s3auth
  `DefaultBucket`/`DefaultHost` expose `Domain.name()/key()/secret()/bucket()/region()` and
  `Bucket.java:16`; rehttp ships `FakeBase`/`FakeStatus` public in `src/main` while genuinely internal
  `DyStatus` is package-private (`DyStatus.java:37`).
- **Interface contract for every public method.** `Base`/`Status` (requs), `Host`/`Bucket`/`Domain`
  (s3auth), `Take`/`Response` (takes), `Scalar`/`Text` (cactoos), with nested `Wrap`/`Simple`
  decorators (`Violation.Simple`, `XeFacet.Wrap`).

## 5. Decoration, inheritance and composition

- **B5.1 Use composition (encapsulation) over inheritance** — Reuse behavior by injecting/wrapping
  collaborators, not by extending classes. Why: inheritance plus virtual methods makes object
  relations complex and lets a parent call child code in reverse (counter-intuitive).
  Example (Java):
  ```java
  final class EncryptedDocument implements Document {
      private final Document plain;
      EncryptedDocument(Document doc) { this.plain = doc; }
      @Override public int length() { return this.plain.length(); }
      @Override public byte[] content() { return decrypt(this.plain.content()); }
  }
  ```

- **B5.2 Inherit only to REFINE an abstract class, never to EXTEND a concrete one** — Refining means
  completing incomplete behavior; extending means intruding on a solid object. Why: a final class is
  a black box you must not modify; an abstract class explicitly invites you to fill its blanks.
  Example (Java):
  ```java
  abstract class Document {
      public abstract byte[] content();
      public final int length() { return this.content().length; }   // final: cannot be overridden
  }
  final class DefaultDocument extends Document { @Override public byte[] content() { return new byte[0]; } }
  final class EncryptedDocument extends Document { @Override public byte[] content() { return decrypt(new byte[0]); } }
  ```

- **B5.3 A class must be `final` or `abstract`** — No middle ground; make intent explicit. Why: a
  neither-final-nor-abstract class may be treated as glass while assuming it is solid, turning method
  overriding into a maintainability hazard.
  Example (Java):
  ```java
  final class Document { public int length() { return 0; } public byte[] content() { return new byte[0]; } }
  interface DocumentContract { int length(); byte[] content(); }
  ```

- **B5.4 Prefer inheritance-free design: extract an interface and wrap** — When you need a variant
  (e.g. `EncryptedDocument`), make the original final, extract an interface, and decorate. Why: this
  keeps the parent solid and lets the variant wrap the default implementation.
  Example (Java):
  ```java
  interface Document { int length(); byte[] content(); }
  final class DefaultDocument implements Document { @Override public int length() { return 0; } @Override public byte[] content() { return new byte[0]; } }
  final class EncryptedDocument implements Document {
      private final Document plain;
      EncryptedDocument(Document doc) { this.plain = doc; }
      @Override public int length() { return this.plain.length(); }
      @Override public byte[] content() { return decrypt(this.plain.content()); }
  }
  ```

- **B5.5 Use composable decorators as the default shape of object-oriented code** — Decorators are
  objects that wrap other objects; compose them into multi-layer structures. Why: the result reads
  declaratively — it declares what a name is without describing how it is built.
  Example (Java):
  ```java
  names = new Sorted<>(
      new Unique<>(
          new Capitalized<>(
              new Replaced<>(
                  new FileNames(new Directory("/var/users/*.xml")),
                  "CL*.+)\\.xml", "$1"
              ))));
  ```

- **B5.6 A decorator's state mirrors the state of what it wraps** — A decorator adds behavior to the
  encapsulated object; it may expose the same interface or a different one. Why: behavior is entirely
  motivated by the wrapped object, so decorators compose cleanly and can even change the represented
  type (`FileNames` iterates strings over an iterable of files).
  Example (Java):
  ```java
  final class Unique<T> implements Iterable<T> {
      private final Iterable<T> origin;
      Unique(Iterable<T> src) { this.origin = src; }
      @Override public Iterator<T> iterator() { /* dedupe underlying iterator */ return this.origin.iterator(); }
  }
  ```

- **B5.7 A "smart" class extends usability; a decorator strengthens an existing method** — Both are
  nested in the interface, but a smart class ADDS methods while a decorator overrides existing ones.
  Why: keep the interface short and cohesive; put shared convenience in `Smart`, enhanced behavior in
  a wrapper (e.g. `Exchange.Fast` adds `toUsd()` and optimizes `rate()`).
  Example (Java):
  ```java
  interface Exchange {
      float rate(String origin, String target);
      final class Fast implements Exchange {
          private final Exchange origin;
          Fast(Exchange exch) { this.origin = exch; }
          @Override public float rate(String source, String target) { return source.equals(target) ? 1.0f : this.origin.rate(source, target); }
          public float toUsd(String source) { return this.origin.rate(source, "USD"); }
      }
  }
  ```

- **B5.8 Avoid the Builder Pattern; it grows objects** — A builder is used to avoid many ctor
  arguments, but many arguments are the real problem; split the object instead. Why: builders push
  toward bigger, less cohesive, less maintainable objects. (The `with*` naming is only acceptable in
  the rare builder you cannot avoid.)
  Example (Java):
  ```java
  // discouraged
  new Book().withAuthor("A").withTitle("T").withPage(page);
  // preferred: small immutable collaborators
  new Book(new Author("A"), new Title("T"), new Page(page));
  ```

- **B5.9 Compose rather than branch: wrap conditionals in objects** — Replace `if`/`for`/`switch`/
  `while` with `If`, `For`, `Switch`, `While` objects. Why: composition of objects is declarative and
  reusable; procedural operators are not composable.
  Example (Java):
  ```java
  // imperative // wrong
  float rate; if (client.age() > 65) { rate = 2.5f; } else { rate = 3.0f; }
  // declarative // right
  float rate = new If(new GreaterThan(new AgeOf(client), 65), 2.5f, 3.0f);
  ```

- **B5.10 Never use static methods as reusable components** — Static methods cannot be passed to a
  ctor, so they cannot participate in composition or decoupling. Why: once statics appear, clean
  object composition becomes impossible and the codebase drifts to imperative/procedural style.
  Example (Java):
  ```java
  // wrong
  static int between(int l, int r, int x) { return Math.min(Math.max(l, x), r); }
  // right
  final class Between implements Number {
      private final Number num;
      Between(Number left, Number right, Number x) { this.num = new Min(new Max(left, x), right); }
      @Override public int intValue() { return this.num.intValue(); }
  }
  ```

- **B5.11 Encapsulate wrapped library calls in a class to isolate static dependencies** — Wrap a
  third-party static utility in a small object so the static call exists in exactly one place. Why:
  this localizes ("isolates the tumor") and lets you remove the dependency incrementally.
  Example (Java):
  ```java
  class FileLines implements Iterable<String> {
      private final File file;
      FileLines(File file) { this.file = file; }
      @Override public Iterator<String> iterator() {
          return Arrays.asList(FileUtils.readLines(this.file)).iterator();
      }
  }
  Iterable<String> lines = new FileLines(f);
  ```

- **B5.12 Decoration + injection is the alternative to inheritance-based frameworks** — Because
  classes are final/abstract and immutability is mandatory, inheritance becomes rare; only
  refinement of abstract classes remains. Why: this forces encapsulation as the main reuse mechanism
  and prevents abusive deep hierarchies.
  Example (Java):
  ```java
  abstract class Document { public abstract byte[] content(); public final int length() { return content().length; } }
  final class DefaultDocument extends Document { @Override public byte[] content() { return new byte[0]; } }
  ```


### Additions from articles

- **Acoa.21 Replace utility methods with composable decorators over one interface** -> add to §5:
  `Text` + `TextInFile`, `PrintableText`, `AllCapsText`, `TrimmedText` compose into one object whose
  behavior is declared but not executed until `read()`. Why: `String.trim().toUpperCase().split()` is
  imperative and runs immediately; an ideal interface has only irreducible methods and everything
  else is decorators. Example (Java):
  ```java
  final Text text = new AllCapsText(
    new TrimmedText(new PrintableText(new TextInFile(new File("/tmp/a.txt"))))
  );
  String content = text.read(); // nothing ran before this line
  ```

- **Acoa.22 Choose vertical vs horizontal decorating deliberately** -> add to §5: Vertical decorating
  nests implementations of one interface (`new Sorted(new Unique(new Odds(...)))`); horizontal puts a
  `Diff[]` array inside a `Modified` object. Why: start vertical for a few decorators, migrate to
  horizontal when their number grows. Example (Java):
  ```java
  interface Diff { Iterable<Integer> apply(Iterable<Integer> origin); }
  Numbers numbers = new Modified(
    new ArrayNumbers(new Integer[] {-1, 78, 4, -34, 98, 4}),
    new Diff[] {new Positive(), new Odds(), new Unique(), new Sorted()}
  );
  ```

- **Acoa.24 Guard abstract classes by making all non-extension methods `final`** -> add to §5: An
  abstract class may expose `abstract` hooks but must `final` everything else (e.g. `read()` final,
  `isValid()` abstract). Why: this is the only inheritance-safe form; otherwise use `final` classes
  with decoration. Example (Java):
  ```java
  abstract class ValidatedHTTPStatus implements Status {
    public final int read() throws IOException { /* uses isValid() */ }
    protected abstract boolean isValid();
  }
  ```
  *(also Apa.18)*

- **Acoa.25 Reject Builder, Facade, Iterator, Mediator, Template Method, Visitor, Singleton** -> add
  to §5/§1: Builder encourages big complex objects (refactor so constructors suffice); Iterator is
  mutable (prefer immutable cursors); Facade/Visitor/Servant are procedural; Template Method relies
  on inheritance. Why: a pattern that needs a builder or a shared mediator is a smell that objects
  are too big. No snippet needed.

- **Acob.26 Prefer composable decorators over traits and mixins** -> add to §5: mixins/traits inject code that reaches into private fields and assume a class's internal structure; decorators layer behavior while keeping objects small and cohesive. Why: Ruby mixins override `to_s` and access `@title`, tearing encapsulation; `new AllCapsText(new TrimmedText(...))` composes externally.
  ```java
  Text text = new AllCapsText(new TrimmedText(new PrintableText(new TextInFile(file))));
  ```

- **Acob.27 Decorate to add behavior without touching the original** -> add to §5: implement the same interface, hold the origin as a `final` field, and forward calls; every concern (timestamping, logging, status, content type) becomes its own small class. Why: `TimedLog`, `StLogged`, `RsWithType`, `RsWithStatus` each add one thing and compose freely.
  ```java
  final class StLogged implements Shift {
    private final Shift origin;
    @Override public Document apply(Document before) {
      Document after = this.origin.apply(before);
      System.out.println("Transformation completed!");
      return after;
    }
  }
  ```
  *(also Adt.16, Apa.16)*

- **Acob.28 Subtyping via interface extension is legitimate** -> add to §5: allow `interface Article extends Manuscript` because it derives a characteristic and preserves LSP; prohibit `class Article extends Manuscript` that copies implementation. Why: the author interviewed David West who said implementation inheritance shouldn't exist in OOP at all; only subtyping fits polymorphism.
  ```java
  interface Manuscript { void print(Console console); }
  interface Article extends Manuscript { void submit(Conference cnf); }
  ```

- **Acob.29 Replace fluent chains with decorator nesting** -> add to §5: compose objects bottom-up (each a small decorator) rather than chaining methods on one bloated object; accept less IDE auto-complete in exchange for isolated, testable classes. Why: adding `multipartBody()` or `timeout()` to a fluent `Request` means editing already blown-up interfaces/classes; a new decorator is a new class.
  ```java
  String html = new BodyOfResponse(
      new ResponseAssertStatus(
          new RequestWithMethod(new JdkRequest("https://www.google.com"), "GET"), 200)
  ).toString();
  ```

- **Acob.30 Decorators legitimately need access to the origin's privates** -> add to §5: consider allowing decorators access to the private attributes of the object they wrap (the author's `trust` idea, later dropped in favor of "decorators see all privates"). Why: `TempFahrenheit` must compute `this.origin.t * 1.8 + 32`; making `t` public breaks encapsulation for everyone, so scoped access to decorators keeps knowledge localized.
  ```java
  // concept: TempCelsius grants its decorators access to private int t
  final class TempFahrenheit implements Temperature {
    private final TempCelsius origin;
    @Override public String toString() { return String.format("%d F", this.origin.t * 1.8 + 32); }
  }
  ```

- **Atq.7 Use decorators for cross-cutting behavior instead of try/catch loops** -> add to §5: Wrap a method/object with a retry decoration (`@RetryOnFailure`) rather than hand-writing `while/try/catch/sleep` inside it. Why: it keeps the object's logic free of recovery plumbing and makes the retry policy a separate, testable concern. Example (Java): see `jcabi-aspects` `@RetryOnFailure(attempts=3, delay=10, unit=TimeUnit.SECONDS)`.

- **Adt.17 Build responses (and objects) by stacking small decorators** -> add to §5: Compose output via nested decorators, each adding one thing, rather than one large builder. Why: small, cohesive, independently testable pieces. Example:
  ```java
  new RsWithStatus(
    new RsWithType(new RsWithBody("<html>Hello, world!</html>"), "text/html"),
    200
  );
  ```

- **Adt.18 Move X-printing behavior into a decorator of the object** -> add to §5: Instead of JAXB/`Marshaller` printing a POJO from outside, create `XmlBook implements Book` that adds `toXML()`. Why: only the object knows how to print itself; external marshalling tears encapsulation.

- **Ada.18 Reject implementation inheritance; extend by decoration** -> add to §5: Do not use inheritance as the extension mechanism; use `final` classes and decorators/envelopes that wrap an interface. Why: the "O" of SOLID explicitly recommends implementation inheritance, which is an anti-OOP technique that cannot be decorated or mocked and ties the subclass to the parent's internals. Example (Java):
  ```java
  interface DB { long cell(String sql); }
  final class Logged implements DB {
    private final DB origin;
    Logged(final DB db) { this.origin = db; }
    public long cell(final String sql) { /* log then delegate */ return origin.cell(sql); }
  }
  ```
  *(also Acs.19, Acoa.23)*

- **Ada.19 Make a custom `Iterator` a read-only adapter over a data source** -> add to §5: Adapt a `Data`-style source to `Iterator<Byte>` by buffering in a queue, adding an exception on `next()` when empty and an `UnsupportedOperationException` in `remove()`. Why: `Iterator` is a fundamental interface, and a correct adapter is the canonical example of wrapping another object to change its protocol. Example (Java):
  ```java
  final class FluentData implements Iterator<Byte> {
    private final Data data;
    private final Queue<Byte> buffer = new LinkedList<>();
    public FluentData(final Data dat) { this.data = dat; }
    public boolean hasNext() {
      if (this.buffer.isEmpty()) {
        for (final byte item : this.data.read()) { this.buffer.add(item); }
      }
      return !this.buffer.isEmpty();
    }
    public Byte next() {
      if (!this.hasNext()) { throw new NoSuchElementException("Nothing left"); }
      return this.buffer.poll();
    }
    public void remove() { throw new UnsupportedOperationException("It is read-only"); }
  }
  ```

- **Ada.20 Do not try to make an iterator thread-safe** -> add to §5: Leave a streaming adapter unsynchronized and document it as not thread-safe; let callers synchronize one level above. Why: iteration spans multiple calls outside the iterator's scope, so `synchronized` methods cannot prevent `hasNext()`/`next()` races anyway. Example (Java):
  ```java
  // Not thread-safe by design; caller synchronizes:
  synchronized (lock) {
    if (iter.hasNext()) { iter.next(); }
  }
  ```

- **Ada.21 Encapsulate primitives inside a bigger object instead of messaging peers** -> add to §5: To make two objects cooperate, wrap them in a composite object that owns both, rather than exposing each other's data. Why: composition increases maintainability while peer messaging is procedural and leaks data. Example (Java):
  ```java
  final class PrintedOn {
    private final Point point;
    private final Canvas canvas;
    PrintedOn(final Point p, final Canvas c) { this.point = p; this.canvas = c; }
  }
  ```

- **Acs.16 Validate with decorators, not inline checks** -> add to §5: Move null/existence/precondition checks out of the core class into small validating decorators that wrap the interface. Why: keeping a class free of validation keeps it small, cohesive and reusable; validators compose in different combinations and can be omitted when validation is too costly. Example (Java):
  ```java
  final class NoWriteOverReport implements Report {
    private final Report origin;
    NoWriteOverReport(Report rep) { this.origin = rep; }
    @Override public void export(File file) {
      if (file.exists()) {
        throw new IllegalArgumentException("File already exists.");
      }
      this.origin.export(file);
    }
  }
  ```

- **Acs.17 Replace convertible if-then-else forking with a decorator** -> add to §5: If a conditional's branching can be expressed as a decorating wrapper, it must be; leaving it inline is a code smell. Why: the conditional "whether to act" is a different responsibility from the core action; moving it to a decorator makes the core class smaller and more cohesive. Example (Java):
  ```java
  final class QuickTalk implements Talk {
    private final Talk origin;
    QuickTalk(Talk t) { this.origin = t; }
    void modify(Collection<Directive> dirs) {
      if (!dirs.isEmpty()) { this.origin.modify(dirs); }
    }
  }
  ```

- **Acs.18 Do not fear method chaining (the real Law of Demeter)** -> add to §5: `book.pages().last().text()` is legal and object-oriented; the LoD forbids only reaching into attributes/getters (`a.x.hello()`), not asking objects to build new objects. Why: the popular "one dot" reading is wrong — it is a getter prohibition in disguise; objects returned by methods are legitimate arguments, so chaining compositions is fine. Example (Java):
  ```java
  final String text = book.pages().last().text();
  ```

- **Acs.20 Separate parsing/printing via a decorated Template** -> add to §5: Model format/parse as a `Template` interface with `with(key, value)`/`read(key)` and add variants through decorators (`RussianTemplate`, `TimezoneTemplate`); do not use a DTO + utility formatter. Why: the DTO/utility split exposes internal data (`get()` on a `TemporalAccessor`) and cannot be extended; a composed template keeps the domain object fully decoupled, replaceable and polymorphic. Example (Java):
  ```java
  final class RussianTemplate implements Template {
    private final Template origin;
    RussianTemplate(Template t) { this.origin = t; }
    @Override public Template with(String key, Object value) {
      Template t = this.origin.with(key, value);
      if (key.equals("MM")) { t = t.with("MMMM", this.name(value)); }
      return t;
    }
  }
  ```

- **Apa.17 Chain small behaviors so each decides or delegates** -> add to §5: Build a chain of objects of one interface (`PsChain` → `PsFake` → `PsCookie` → `PsFacebook`); each tries in order and the first success wins. Why: a chain of single-purpose objects is extensible and testable, unlike a switch statement. Example:
  ```java
  new PsChain(
      new PsByFlag(new PsByFlag.Pair("PsLogout", new PsLogout())),
      new PsCookie(codec));
  ```

- **Apb.10 Decorate the small object, not the container** -> add to §5: When extending behavior
  (e.g. logging on delete), decorate the individual retrieved object rather than the whole collection.
  Why: the decoratee is smaller and more focused, so the decorator is more cohesive and honors the
  open-closed principle without touching Book or Books. Example:
  ```text
  b = Logged.new(books.findById(42))
  b.remove
  ```

### Additions from real codebases

- **Cactoos: 116 classes `extends …Envelope`; delegate methods are `final`.** `ScalarEnvelope`
  (`scalar/ScalarEnvelope.java:17-36`), `TextEnvelope` (`text/TextEnvelope.java:14-48`). Because the
  delegate methods are `final`, subclasses cannot break the base contract — the fragile-base-class
  problem is designed away (`README.md:388-395`).
- **Takes: `Wrap` base classes are the deliberate extension seam.** `RsWrap`/`TkWrap` are concrete
  `public class` (not `final`, not `abstract`) with `final` delegate methods, purely so decorators can
  `super(...)` a composed origin (`rs/RsWrap.java:28-52`, `tk/TkWrap.java:68-89`); decorators are named
  prefix+preposition (`TkWithHeader`, `RsWithCookie`, `RqWithAuth`, `TkGzip`, `BkParallel`).
- **Qulice's inheritance gates.** `DesignForExtension`, `FinalClass`, `ProtectedMethodInFinalClassCheck`,
  `RedundantSuperConstructorCheck`, `InnerTypeLast`, `DeclarationOrder`, `NoClone`, `NoFinalizer`,
  `IllegalType`/`IllegalImport` (`checks.xml:212-253,281,487,504`).
- **s3auth: decoration over inheritance for every concern.** `FastHost` (`@Timeable`), `SmartHost`,
  `RejectingHost`, `GzipResource`, `SyslogResource`, `SecuredHost`, `DynamoHosts`, `SyslogHosts` all
  implement the same interface and hold a `private final transient` origin.
- **AOP as the "method decorator" where subclassing would break EO.** jcabi-aspects `@Cacheable`,
  `@Loggable`, `@Timeable`, `@RetryOnFailure` (s3auth, requs). Use sparingly: the book rejects
  behavior-carrying annotations (`Adt.2`) while `Adt.65` allows weaving only as a reluctant fallback.
- **Lombok for value semantics.** Takes uses `@ToString(of="origin")` + `@EqualsAndHashCode` on
  `TkWrap`/`RsWrap` and `@EqualsAndHashCode(callSuper=true)` on `BkParallel`/`TkGzip`; xembly
  `@EqualsAndHashCode(of="name")`.

## 6. Error handling, exceptions and null

- **B6.1 Never accept `null` as a method argument** — Design methods (and interfaces like `Mask`)
  so callers always pass a real object; use a Null Object when "nothing" is intended. Why: accepting
  `null` forces `mask == null` checks, treats the object as disabled data, and shifts responsibility
  away from it.
  Example (Java):
  ```java
  // wrong
  Iterable<File> find(String mask) { if (mask == null) { /* all */ } else { /* ... */ } return null; }
  // right: always pass an object
  interface Mask { boolean matches(File file); }
  class AnyFile implements Mask { @Override public boolean matches(File file) { return true; } }
  Iterable<File> find(Mask mask) { /* mask always non-null */ return java.util.Collections.emptyList(); }
  ```

- **B6.2 Do not defend against a `null` that arrives anyway** — Do not add `if (x == null) throw ...`
  guards; let the JVM raise `NullPointerException` when the argument is used. Why: defensive checks
  pollute the code; NPE is the standard, proper indicator of a wrongly passed `null`. (An explicit
  `IllegalArgumentException` is the book's described "defensive" alternative, but the "ignorant"
  approach is preferred.)
  Example (Java):
  ```java
  // preferred: ignore and let NPE happen
  Iterable<File> find(Mask mask) { /* manipulate mask; NPE is fine */ return java.util.Collections.emptyList(); }
  ```

- **B6.3 Never return `null`** — Returning `null` betrays the caller and forces trust-destroying
  checks (`if (title == null)`). Why: an object is either alive or dead; there is no third state. Use
  one of the three alternatives: throw, return a collection, or return a Null Object.
  Example (Java):
  ```java
  // wrong
  String title() { if (/* none */) { return null; } return "Elegant Objects"; }
  // right (collection)
  Collection<String> titles() { return java.util.Collections.emptyList(); }
  ```

- **B6.4 Prefer fail-fast over fail-safe** — Crash at the first sign of a problem; make the software
  fragile and cover it with tests. Why: hiding a problem (returning `null`/`0`/"safe" defaults)
  postpones the crash to a far, hard-to-debug place; revealing it early speeds every fix.
  Example (Java):
  ```java
  // fail-safe // wrong
  int length(File file) { try { return content(file).length; } catch (IOException ex) { return 0; } }
  // fail-fast // right
  int length(File file) throws IOException { return content(file).length; }
  ```

- **B6.5 Throw only checked exceptions** — Use checked exceptions so the compiler forces callers to
  handle or declare them; unchecked exceptions hide the failure mode from the signature. Why: checked
  exceptions are visible and transfer responsibility explicitly; unchecked ones are invisible in the
  contract. (The book calls unchecked exceptions a mistake; multiple exception types are also
  discouraged — see B6.9.)
  Example (Java):
  ```java
  public byte[] content(File file) throws IOException { /* ... */ return new byte[0]; }
  ```

- **B6.6 Don't catch unless you have to; escalate instead** — Every `catch` needs a strong reason;
  let exceptions float to the top. Why: "rescuing" everywhere puts you in fail-safe mode; ideally
  there is one catch per application entry point.
  Example (Java):
  ```java
  // preferred: just declare and rethrow
  public int length(File file) throws IOException { return content(file).length; }
  ```

- **B6.7 Always chain exceptions; never swallow the root cause** — When you catch and rethrow, wrap
  the original as the cause (`new Exception(msg, ex)`). Why: chaining preserves the low-level root
  cause while enriching the context at each level; dropping `ex` loses hours of debugging.
  Example (Java):
  ```java
  // right
  public int length(File file) throws Exception {
      try { return content(file).length; }
      catch (IOException ex) { throw new Exception("Can't calculate file length.", ex); }
  }
  // wrong: ignores the cause
  // catch (IOException ex) { throw new Exception("Can't calculate it"); }
  ```

- **B6.8 Recover only once, at the top-level entry point** — Catch-and-resolve exactly at the
  application boundary; everywhere else catch→chain→rethrow. Why: recovery in the middle is "using
  exceptions for flow control" and hides problems; one recovery point keeps the whole system clean.
  Example (Java):
  ```java
  public final class App {
      public static void main(String... args) {
          try { System.out.println(new App().run()); }
          catch (Exception ex) { System.err.println("I'm sorry, there was a problem: " + ex.getLocalizedMessage()); }
      }
      String run() { return ""; }
  }
  ```

- **B6.9 One exception type is enough** — Do not create many exception types; if you only ever chain
  and rethrow, the type is never used. Why: with a single recovery point and no flow-control catches,
  the chained exception carries the full context; type information is redundant.
  Example (Java):
  ```java
  // preferred: a single checked type carried up through chained causes
  public int length(File file) throws Exception { return content(file).length; }
  ```

- **B6.10 Do not catch-and-log** — Logging an exception and continuing is a fail-safe anti-pattern.
  Why: it conceals the failure and makes the eventual crash impossible to trace; escalate or chain
  instead.
  Example (Java):
  ```java
  // anti-pattern
  try { content(file); } catch (IOException ex) { log.error("oops", ex); }
  // right: chain and rethrow, or don't catch at all
  ```

- **B6.11 Use AOP for cross-cutting retries** — Retry logic (e.g. `@RetryOnFailure(attempts = 3)`)
  belongs in an aspect, not inside the method. Why: retrying requires catching/recovering in the
  middle, which otherwise contradicts fail-fast; an aspect moves that mechanism out of the main
  classes and removes verbosity/duplication.
  Example (Java):
  ```java
  @RetryOnFailure(attempts = 3)
  public String content() throws IOException { return http(); }
  ```

- **B6.12 Do not return sentinel values (`-1`, `0`) in place of NULL** — A scalar sentinel is
  semantically identical to `null` and forces a mistrusting `== -1` check. Why: the caller must
  remember the sentinel or misread it as a real value, producing unpredictable bugs.
  Example (Java):
  ```java
  // wrong
  int length(File file) { return -1; }
  // right: throw, or return a collection / Null Object
  int length(File file) throws IOException { return content(file).length; }
  ```

- **B6.13 Implement the Null Object pattern when "not found" is routine** — Return an object of the
  same interface that answers benignly and throws on operations that make no sense. Why: it keeps the
  caller's trust and type intact (unlike `Optional`, which the book rejects as counter-OOP).
  Example (Java):
  ```java
  final class NullUser implements User {
      private final String label;
      NullUser(String name) { this.label = name; }
      @Override public String name() { return this.label; }
      @Override public void raise(Cash salary) { throw new IllegalStateException("You can't raise my salary, I'm a stub"); }
  }
  ```

- **B6.14 Avoid `java.util.Optional`** — Do not use `Optional` to signal absence. Why: the method name
  (`user()`) then lies — it returns an envelope, not a user; it is semantically close to `null` and
  against object thinking. Prefer a collection or a Null Object.
  Example (Java):
  ```java
  // discouraged
  java.util.Optional<User> user(String name) { return java.util.Optional.empty(); }
  // preferred
  Collection<User> users(String name) { return java.util.Collections.emptyList(); }
  ```

- **B6.15 Never use exceptions for flow control** — Do not catch an exception to branch/recover
  (`age = -1` in a `catch`). Why: exceptions signal critical, non-recoverable situations; using them
  to fork is the same anti-pattern as returning `null`.
  Example (Java):
  ```java
  // anti-pattern
  int age; try { age = Integer.parseInt(text); } catch (NumberFormatException ex) { age = -1; }
  ```

- **B6.16 Catch only to chain and rethrow** — The sole legal in-flight catch purpose is wrapping and
  rethrowing. Why: if you never use exception types for decisions and never recover early, the only
  remaining job of a catch is to enrich the context.
  Example (Java):
  ```java
  try { return content(file).length; }
  catch (IOException ex) { throw new Exception("Can't calculate file length.", ex); }
  ```


### Additions from articles

- **Acoa.26 Replace `null` with a Null Object constant or a thrown exception** -> add to §6:
  `Employee.NOBODY` (a constant Null Object) or `throw new EmployeeNotFoundException(name)`; make
  methods "extremely demanding" and let them fail fast. Why: `null` forces ad-hoc `if (x == null)`
  checks and fails slowly, hiding the failure. Example (Java):
  ```java
  public Employee getByName(String name) {
    int id = database.find(name);
    if (id == 0) { throw new EmployeeNotFoundException(name); }
    return new Employee(id);
  }
  ```

- **Acoa.27 Let a Null Object throw on entity-specific calls** -> add to §6: An anonymous Null
  `Employee` can answer common calls (`name()` -> "anonymous") but `throw` on `transferTo(...)`. Why:
  this exposes common behavior and refuses only operations that make no sense for the null case.
  Example (Java):
  ```java
  if (id == 0) {
    employee = new Employee() {
      public String name() { return "anonymous"; }
      public void transferTo(Department dept) {
        throw new AnonymousEmployeeException("I can't be transferred, I'm anonymous");
      }
    };
  }
  ```

- **Acoa.28 Recognise `Map.get()` returning null as a design flaw and prefer a finder/Iterator** ->
  add to §6: A `find()` returning an `Iterator`/cursor lets you check `hasNext()` without the double
  lookup or the null. Why: `containsKey` + `get` searches twice; `get`-returns-null forces a null
  check. Example (Java):
  ```java
  Iterator found = Map.search("Jeffrey");
  if (!found.hasNext()) { throw new EmployeeNotFoundException(); }
  return found.next();
  ```

- **Acoa.29 Eliminate `if (x != null)` in `finally` with a final local or a second try/catch** -> add
  to §6: Declare the resource `final` and open it before `try`; when opening itself throws, use two
  `try`/`catch` blocks. Why: the presence of `null` is a code smell; only third-party/JDK APIs that
  return `null` justify `if (x == null)`. Example (Java):
  ```java
  final InputStream input;
  try { input = url.openStream(); }
  catch (IOException ex) { throw new RuntimeException(ex); }
  try { /* read, throws IOException */ }
  catch (IOException ex) { throw new RuntimeException(ex); }
  finally { input.close(); }
  ```

- **Acob.31 Replace NULL/exception-on-empty with a default argument** -> add to §6: when a lookup may find nothing, accept a "default" object to return instead of returning NULL, false, NaN, or throwing. Why: languages disagree (`Collections.max` throws, Ruby returns nil, PHP false, JS NaN); `max(list, def)` is better than an exception, and Python's `max()` already does this.
  ```java
  Integer max(List<Integer> items, Integer def) {
    return items.isEmpty() ? def : Collections.max(items);
  }
  ```

- **Acob.32 Never use reflection; it means hidden coupling** -> add to §6: forbid `instanceof`, casting, `Class.forName`, annotations, serialization engines, and reflection-based test tricks because they couple code invisibly and fail only at runtime. Why: `((Collection) items).size()` makes `calc()` depend on `sizeOf`'s implementation; when `sizeOf` later handles any `Iterable`, `calc` still guards with `instanceof` and drifts out of sync.
  ```java
  // avoided:
  // Method m = book.getClass().getDeclaredMethod("name"); m.setAccessible(true);
  // preferred: inject System.out (or a Console) and test print()
  ```

- **Acob.33 Inject the log; default to a no-op `Log::NULL`** -> add to §6: pass a log dependency into objects rather than using a static/global logger; default it to a logger that prints nowhere. Why: a global logger makes it hard to assert logging in tests and to separate user-facing log lines in a web/multi-threaded context; an injected `log` is fakeable in unit tests.
  ```ruby
  class Zold::List
    def initialize(wallets:, log: Log::NULL)
      @wallets = wallets
      @log = log
    end
    def run
      @wallets.all.sort.each { |id| @log.info("#{id}") }
    end
  end
  ```

- **Atq.8 Never catch an exception without re-throwing** -> add to §6: `catch` only to chain/rethrow with more context; printing a stack trace and swallowing breaks the trust chain. Why: an exception is a bubble carrying information that must float to the top, not be popped in place. Example (Java):
  ```java
  final class Wire {
    public void send(final int data) throws IOException {
      this.stream.write(data);
    }
  }
  ```
  *(also Adt.19)*

- **Atq.9 Prefer checked exceptions, one type, no unchecked** -> add to §6: Declare `throws Exception` (single type), never throw/catch `RuntimeException` subtypes, and never hide failure behind unchecked exceptions. Why: a method's signature should reveal that it is unsafe; unchecked exceptions mask that and let methods grow into multi-purpose messes. Example (Java): `public void save(File file, byte[] data) throws Exception;`.
  *(also Apa.19)*

- **Atq.10 Fail fast instead of returning a safe default** -> add to §6: Throw on missing/invalid input rather than returning `0`, `null`, or an empty collection; make failures loud and visible. Why: fail-safe concealment makes bugs hard to spot and the code hard to stabilize. Example (Java):
  ```java
  public int size(File file) {
    if (!file.exists()) {
      throw new IllegalArgumentException(
        String.format("File %s doesn't exist", file.getAbsolutePath())
      );
    }
    return (int) file.length();
  }
  ```
  *(also Acob.34)*

- **Atq.11 Put maximum context in every thrown/re-thrown message** -> add to §6: An exception message must name the concrete object/identifier involved; never throw bare `"File doesn't exist"`. Why: production logs without context make bugs impossible to reproduce; a vague message is disrespect to the caller. Example (Java):
  ```java
  throw new IllegalArgumentException(
    String.format("User profile file %s doesn't exist", file.getAbsolutePath())
  );
  ```

- **Atq.12 One catch block per exception originator** -> add to §6: Don't group exceptions from several calls into one `catch`; wrap each originator separately with its own message. Why: a shared catch loses which call failed and cannot produce a focused message. Example (Java):
  ```java
  URL url;
  try {
    url = new URL(uri);
  } catch (MalformedURLException ex) {
    throw new IllegalArgumentException(
      String.format("Failed to parse the URI '%s'", uri), ex
    );
  }
  ```

- **Atq.13 Keep try-blocks as small as the throwing call** -> add to §6: A `try` must wrap only the statement that can actually throw; move the rest out. Why: a wide block can only emit an inaccurate error message and hides unrelated statements from the reader. Example (Java):
  ```java
  String[] lines;
  try {
    lines = Files.readAllLines(file);
  } catch (IOException ex) {
    throw new IllegalStateException(
      String.format("Failed to read all lines from %s", file), ex
    );
  }
  for (final String line : lines) { ... }
  ```

- **Atq.14 Restore the interrupt flag before rethrowing** -> add to §6: When catching `InterruptedException`, call `Thread.currentThread().interrupt()` before wrapping/throwing. Why: `Thread.interrupted()` clears the flag, so swallowing it loses the owner's stop request forever. Example (Java):
  ```java
  try {
    Thread.sleep(100);
  } catch (InterruptedException ex) {
    Thread.currentThread().interrupt();
    throw new IllegalStateException(ex);
  }
  ```

- **Atq.15 Never use Java `assert`** -> add to §6: Use exceptions (or a test matcher) instead of `assert`; assertions are disabled by default and extend `Error`, so production never sees them. Why: fail-fast demands bugs be visible in production too. Example: replace `assert x > 0 : "..."` with `if (x <= 0) throw new IllegalArgumentException("...");`.

- **Atq.16 Retry transient operations via a policy, not ad-hoc loops** -> add to §6: For JDBC selects, HTTP/S3/FTP loads and REST calls, apply a bounded retry with a delay before finally throwing. Why: occasional failures should not surface immediately, but retries must be explicit and capped. Example (Java): `@RetryOnFailure(attempts = 3, delay = 10, unit = TimeUnit.SECONDS)` on `load(URL)`.

- **Adt.20 Fail on non-zero exit codes with a `Safe` decorator** -> add to §6: Wrap a `Shell` in `Shell.Safe` so a non-zero exit throws instead of silently returning. Why: it centralizes `if (code != 0) throw` instead of duplicating it, and makes shell failures loud. Example:
  ```java
  Shell ssh = new Shell.Safe(new SSH(host, 22, user, key));
  ```

- **Adt.21 Retry via a decorator, not an annotation** -> add to §6: Implement retry-on-exception as an explicit wrapper (`new FooThatRetries(new Foo())`) around a `Retry` object. Why: the retry loop is a behavior that must be visible in composition; `@RetryOnFailure` hides it behind the weaver.

- **Ada.22 Throw `NoSuchElementException` when an iterator is exhausted** -> add to §6: A `next()` on an empty adapter must fail fast with `NoSuchElementException`, not return null or a sentinel. Why: null-and-sentinel returns hide the exhausted state and violate no-null; a thrown exception makes the contract explicit. Example (Java):
  ```java
  public Byte next() {
    if (!this.hasNext()) {
      throw new NoSuchElementException("Nothing left");
    }
    return this.buffer.poll();
  }
  ```

- **Ada.23 Unsupported operations must throw, not silently no-op** -> add to §6: A method that the object cannot honor (e.g. `remove()` on a read-only iterator) throws `UnsupportedOperationException`. Why: silent no-ops let callers believe an operation succeeded, hiding a contract violation. Example (Java):
  ```java
  public void remove() {
    throw new UnsupportedOperationException("It is read-only");
  }
  ```

- **Ada.24 Wrap checked IO in `IoCheckedScalar`, don't leak it into the design** -> add to §6: Use an IO-checking scalar decorator so callers deal with a uniform interface rather than hand-thrown checked exceptions scattered through constructors. Why: it keeps constructors code-free and centralizes the checked-to-runtime translation. Example (Java):
  ```java
  this.text = new IoCheckedScalar<>(new SyncScalar<>(new StickyScalar<>(source)));
  ```

- **Acs.21 Let methods crash; do not defend inline** -> add to §6: Remove null/existence guards from core methods and let invalid input fail fast with a clear exception; put protection in validating decorators instead. Why: inline validation bloat mixes unrelated concerns into the class, whereas `null` supplied to a core method should crash immediately rather than be silently tolerated. Example (Java):
  ```java
  final class DefaultReport implements Report {
    @Override public void export(File file) {
      // no null / exists checks here; use NoNullReport/NoWriteOverReport
    }
  }
  ```

- **Acs.22 Never write code after a throw; drop else/continuation** -> add to §6: Treat `throw`, `continue` and `return` as dead ends — delete any `else` branch or following statement that assumes execution continues. Why: a branch after a throw is unreachable, so it is at best confusing noise and at worst the buggy `throw` then `System.exit(1)` pattern; a proper control-flow fork has no road past the dead end. Example (Java):
  ```java
  if (x < 0) {
    throw new IllegalArgumentException("X can't be negative");
  }
  System.out.println("X is positive or zero");
  ```

- **Apa.20 When you cannot reproduce a bug, write a passing test that proves it works and close the ticket** -> add to §6: If the failure is unreproducible, add a test asserting correct behavior, commit it, and report the issue resolved with evidence. Why: honest "we could not reproduce; here is proof of intended behavior" is better than a stalled ticket or a disabled feature. Example:
  ```text
  // Test passes; build stays green; ticket closed with the new test as evidence.
  // If it recurs, a new linked bug inherits this investigation.
  ```

- **Apa.21 Disable the toxic feature rather than keep the ticket hostage** -> add to §6: If a production bug cannot be fixed quickly and the test trick fails, disable the feature, ship, and close. Why: a ticket held too long makes the work unmanageable; a temporarily disabled feature is an acceptable, honest trade-off. Example:
  ```text
  // release with feature flag OFF; close ticket; open a follow-up for the root cause
  ```

- **Apa.22 Say "no" early and hand the problem back** -> add to §6: If a task is beyond you, decline immediately instead of stalling; honesty preserves reputation and lets the manager reassign. Why: pretending to solve it damages your reputation, the schedule and the project. Example:
  ```text
  "No, I can't do this; find someone else." — said as soon as you know it.
  ```

- **Apb.11 Return Null/Fake objects, never a boolean** -> add to §6: A retrieval API should return an
  object (possibly a null/fake object) rather than throwing or returning `false`; let the caller act on
  it and re-raise with context. Why: returning `false` violates command-query separation, and a returned
  object enables the Null Object pattern plus flexible error handling. Example:
  ```text
  begin
    books.findById(42).remove
  rescue e
    raise e, "Can't delete book #42"
  end
  ```

- **Apb.12 Kill NULLs as a refactoring step** -> add to §6: Drive `null` usages toward zero, removing
  each one that is not forced by the JDK. Why: NULL is "evil" and a leading cause of defects; in Takes
  it was reduced to only the ~58 unavoidable JDK cases, giving roughly 1 null per 2,700 lines.
  Example:
  ```text
  // keep the 58 cases that come from the JDK; remove every project-owned null.
  ```

### Additions from real codebases

- **Cactoos centralizes every `null` decision in a decorator.** `NoNulls` rejects a null origin with
  `IllegalArgumentException` and a null value with `IllegalStateException` rather than returning null
  (`scalar/NoNulls.java:31-45`). The 117 raw `null` hits in main are concentrated in such
  null-rejecting decorators and Javadoc — not business logic. The trio
  `Unchecked`/`IoChecked`/`Checked` translate exception dialects. Lesson: "no null" means **no null
  propagates**, enforced by objects.
- **Takes models absence with an `Opt<T>` nullable object.** `Single`/`Empty` variants
  (`misc/Opt.java:11-30`). Its README claims "not a single `null`" but main still has ~62 null
  occurrences for internal lazy caches/legacy JDK signatures (`HttpException.java:39`,
  `Options.java:80`, `Href.java:342-381`, `TkFallback.java:213-214`) — treat README marketing as a
  goal to verify, not a fact.
- **Fail fast with context.** s3auth `DefaultHost`'s private `validate()` throws
  `IllegalStateException("The key of the bucket is empty")` (`DefaultHost.java:255-266`); parsing
  throws `SyntaxException`; `Xembler.apply` wraps each failure with its directive position:
  `String.format("Directive #%d: %s", pos, dir)` (`Xembler.java:139-148`).
- **Qulice enforces resource safety.** PMD `CloseInlineResourceRule` forces try-with-resources for
  inline `AutoCloseable`; `CloseInlineResourceRule`/`UnnecessaryLocalRule` are custom rules; tests use
  OO matchers rather than `Assert.assert*`.
- **Deviations to flag, not copy.** requs `Retry.call()` may return `null`, `Compiler` uses
  `assert this.properties != null`; rehttp `MainTest` must swap `System.out` and reconfigure Log4j to
  test a static `main` (`MainTest.java:96-116`) — concrete evidence that static entry points are hard
  to test.

## 7. Testing

- **B7.1 A unit test is part of the class** — Conceptually, the test belongs to the class like its
  methods and name. Why: classes without tests should not ship; a good test makes the class cleaner
  and more maintainable and dramatically reduces the need for documentation.
  Example (Java):
  ```java
  final class CashTest {
      @Test public void summarizes() {
          assertThat(new Cash("$5").plus(new Cash("$3")), equalTo(new Cash("$8")));
      }
  }
  ```

- **B7.2 Write tests instead of documentation** — Unit tests are the documentation; demonstrate usage
  rather than describing it. Why: a test is international and unambiguous, and is read more often than
  the class itself; "don't tell, demonstrate".
  Example (Java):
  ```java
  @Test public void deducts() {
      assertThat(new Cash("$7").plus(new Cash("-$11")), equalTo(new Cash("-$4")));
  }
  @Test public void multiplies() {
      assertThat(new Cash("$2").mul(3), equalTo(new Cash("$6")));
  }
  ```

- **B7.3 A test method has a single `assertThat` and no other statements** — Arrange via constructor
  composition; the only statement in the body is one assertion. Why: single-statement tests are short,
  focused and maintainable; imperative test scripts are fragile and hard to read.
  Example (Java):
  ```java
  @Test public void summarizes() {
      assertThat(new Cash("$5").plus(new Cash("$3")), equalTo(new Cash("$8")));
  }
  ```

- **B7.4 Don't mock; use fakes** — Never use Mockito-style mocks; ship a `Fake` implementation
  nested in the interface and use it in tests. Why: mocks hard-code assumptions about internal
  interactions (turning assumptions into facts), so tests fail on harmless refactors and lose the
  refactoring-safety property.
  Example (Java):
  ```java
  // wrong
  Exchange exchange = Mockito.mock(Exchange.class);
  Mockito.doReturn(1.15).when(exchange).rate("USD", "EUR");
  // right
  Exchange exchange = new Exchange.Fake(1.2345);
  Cash euro = new Cash(exchange, 500).in("EUR");
  ```

- **B7.5 Every interface ships a nested fake** — Attach a `final class Fake implements Interface` to
  each interface. Why: it makes tests short, keeps them independent of implementation details, and
  spreads better testing practice to clients.
  Example (Java):
  ```java
  interface Exchange {
      float rate(String origin, String target);
      final class Fake implements Exchange {
          private final float value;
          Fake(float v) { this.value = v; }
          @Override public float rate(String origin, String target) { return this.value; }
      }
  }
  ```

- **B7.6 Fakes double as interface design review** — Writing a fake forces you to think like a user of
  the interface (where does content live? how is thread safety handled?). Why: answering those
  questions inevitably improves the interface; fakes can be more complex than the real classes.
  Example (Java):
  ```java
  interface WebPage {
      String content();
      void update(String content);
      final class Fake implements WebPage {
          private String text = "";
          @Override public String content() { return this.text; }
          @Override public void update(String content) { this.text = content; }
      }
  }
  ```

- **B7.7 Tests only care about public behavior, never internal interactions** — Do not verify how
  many times a dependency method was called or with what arguments. Why: that is the object's private
  business; testing it couples tests to implementation and makes refactoring impossible.
  Example (Java):
  ```java
  // wrong: verification of interactions
  Mockito.verify(exchange, Mockito.times(1)).rate("USD", "EUR");
  // right: assert the observable result only
  assertThat(euro.toString(), equalTo("6.17"));
  ```

- **B7.8 When an interface changes, its fake changes with it; the test does not** — A fake lives with
  the interface, so refactors that do not change public behavior leave tests green. Why: this is the
  true purpose of a unit test — to fail only on behavior change (no false positives).
  Example (Java):
  ```java
  interface Exchange {
      float rate(String target);
      float rate(String origin, String target);
      final class Fake implements Exchange {
          @Override public float rate(String target) { return this.rate("USD", target); }
          @Override public float rate(String origin, String target) { return 1.2345f; }
      }
  }
  // Existing Cash tests need no change.
  ```

- **B7.9 Treat tests with the same care as production code** — Test code is not second-class; apply
  the same quality standards. Why: tests are the primary maintainability tool; sloppy tests erode
  trust and get abandoned.
  Example (Java):
  ```java
  // Clean, small test classes with the same naming/immutability standards as production.
  ```

- **B7.10 Design for testability via constructor injection** — A class that receives all dependencies
  through its ctor can be tested with fakes; a class that calls `new` internally cannot. Why: hard-
  coded dependencies force live network/DB round-trips in tests, making them slow and untrustworthy.
  Example (Java):
  ```java
  final class Cash {
      private final int dollars; private final Exchange exchange;
      Cash(int value, Exchange exch) { this.dollars = value; this.exchange = exch; }
      int euro() { return (int) (this.exchange.rate("USD", "EUR") * this.dollars); }
  }
  Cash five = new Cash(5, new Exchange.Fake(2.0f));
  ```

- **B7.11 Fail-fast design makes bugs reproducible by tests** — Make the software fragile so every
  break is an obvious control point; then unit tests reproduce it easily. Why: hidden (fail-safe)
  errors are impossible to reproduce reliably; visible ones become tests, and each fix increases
  stability.
  Example (Java):
  ```java
  // prefer throwing over silent fallbacks so tests can assert the failure
  int length(File file) throws IOException { return content(file).length; }
  ```

- **B7.12 Bug fixes ship with a reproducing test** — (Bug-Driven Development) — When a bug is found,
  add a failing test that reproduces it, then fix. Why: fail-fast plus a test per bug creates a
  permanent safety net and prevents regressions. *(EO framing: tests are the safety net that makes
  refactoring and bug-fixing safe.)*
  Example (Java):
  ```java
  @Test public void reproducesBug() {
      assertThat(new Cash("$0").mul(2), equalTo(new Cash("$0")));
  }
  ```


### Additions from articles

- **Acoa.31 Keep objects small so fakes and mocks stay small** -> add to §7: Mocking pain is a symptom
  that objects are too big; a one-method interface (e.g. `Ocket.read`) is trivially mockable. Why:
  the root cause of "mocking is evil" is large objects with many methods returning other objects.
  Example (Java):
  ```java
  Ocket ocket = Mockito.mock(Ocket.class);
  Mockito.doAnswer(inv -> {
    OutputStream.class.cast(inv.getArguments()[0]).write(' ');
    return null;
  }).when(ocket).read(Mockito.any(OutputStream.class));
  ```

- **Acoa.32 Use interfaces (not statics) as the seam that makes code testable** -> add to §7: Static
  utility calls are hard-coded dependencies that can never be broken for tests; an interface can be
  replaced by a fake/mock. Why: `S3Md5Hash(Ocket)` is testable purely because `Ocket` is an interface.
  No new snippet; cite the Ocket mock above.

- **Acob.35 Keep test methods to a single `assertThat`** -> add to §7: move arranging logic into helper classes built by constructors, so the test body is one assertion; use Hamcrest. Why: benefits are reusability (`ArrayFromRandom`), brevity, readability, and it becomes impossible to keep setters in production code because there is no room for algorithmic code.
  ```java
  @Test
  public void testIntStream() {
    final long seed = System.currentTimeMillis();
    assertThat(
      new ArrayFromRandom(new Random(seed)).toArray(SIZE),
      equalTo(new Random(seed).ints().limit(SIZE).toArray())
    );
  }
  ```
  *(also Atq.28)*

- **Acob.36 Recognize the canonical unit-testing anti-patterns** -> add to §7: audit tests against the full list — Cuckoo, Test-per-Method, Anal Probe, Conjoined Twins, Happy Path, Slow Poke, Giant, Mockery, Inspector, Generous Leftovers, Local Hero, Nitpicker, Secret Catcher, Dodger, Loudmouth, Greedy Catcher, Sequencer, Enumerator, Free Ride, Excessive Setup, Line Hitter, Forty-Foot Pole Test, The Liar. Why: each names a concrete maintainability defect (e.g. Inspector breaks on refactor; Line Hitter hits 100% coverage without analyzing output).
  ```java
  // Test-per-Method: one test per production method — almost always a bad idea.
  // Happy Path: no boundary/exception cases (e.g. -2 years old).
  ```

- **Acob.37 Reflective tests destroy the safety net** -> add to §7: never reach private methods via reflection, even to test them; make the collaborator injectable instead. Why: after changing `name()` to return `StringBuilder` (a legal refactor), a reflection-based test fails falsely; the author then cannot trust green tests. This is the "Inspector" anti-pattern.
  ```java
  // made testable instead: inject a Console so print() can be asserted
  ```

- **Atq.17 Plan a target number of bugs as the test exit criterion** -> add to §7: Stop testing when a forecast number of defects has been found, not when "no bugs remain". Why: defects are unlimited, so testing can never prove correctness; a forecast count is the only measurable, payable criterion. Example: for a project like a prior one that produced 500 bugs, set the goal "discover 500 bugs" before shipping.

- **Atq.18 Test to find errors, not to confirm correctness** -> add to §7: Frame every test as an attempt to break the product; an unsuccessful test is one that found no error. Why: Myers's principle — testing is a destructive process — is the foundation of effective testing. Example: prefer an assertion that would fail on a plausible bug over a smoke test that always passes.

- **Atq.19 A missing test is a bug** -> add to §7: File a bug when a class has no unit test or an existing test doesn't cover a critical aspect. Why: uncovered code is the first to break under refactoring and silently ships regressions. Example: report "`Metrics` has no test for zero-file input" as a defect, not a task.

- **Atq.20 A flaky test is a bug** -> add to §7: File a defect for any test that fails sporadically or only in some environments. Why: an unstable test erodes the safety net and trains the team to ignore red builds. Example: "`VerboseListTest` fails intermittently on CI" is a ticket.

- **Atq.21 No `@Before`/`@BeforeClass`, no fixtures** -> add to §7: Build all needed data inside each test method; never share setup through JUnit lifecycle hooks. Why: shared fixtures couple test methods so changing one breaks others. Example (Java): see Atq.23.

- **Atq.22 Test methods share nothing — no shared literals or fields** -> add to §7: Give each test method its own local constants/objects; different tests should use different literals, even at the cost of "duplication". Why: a shared `private static final MSG` ties unrelated tests together and forces coordinated edits. Example (Java):
  ```java
  @Test
  void simplyWorks() {
    final String msg = "something";
    assertThat(new Foo(msg).doSomething(), containsString(msg));
  }
  ```
  *(also Adt.24)*

- **Atq.23 Build test scaffolding with fake objects in `src/main`, not static helpers** -> add to §7: Ship reusable test fixtures as `Fk*`/factory classes next to production code, with a real ctor API. Why: fake objects are reusable across test classes and can themselves be tested; private static helpers are utility anti-patterns. Example (Java):
  ```java
  final class FkFolder implements Folder, Closeable {
    private final File dir;
    private final String[] parts;
    public FkFolder(String... prts) {
      this(Files.createTempDirectory("test-1"), prts);
    }
    @Override
    public Iterable<File> files() {
      final Folder folder = new DiscFolder(this.dir);
      for (final String part : this.parts) {
        final String[] pair = part.split(":", 2);
        folder.save(pair[0], pair[1]);
      }
      return folder.files();
    }
    @Override
    public void close() { FileUtils.deleteDirectory(this.dir); }
  }
  ```
  *(also Acoa.30)*

- **Atq.25 Design code so tests can easily break it** -> add to §7: A codebase should be trivial to make red; a passing test is a weak test. Why: if you optimize code to satisfy unwritten future tests, tests become useless green light. Example: prefer small noun-objects with injectable collaborators over static methods that are hard to falsify.

- **Atq.26 Test your test doubles** -> add to §7: If a fake/double is unreliable, verify it too; don't assume the double is correct. Why: broken doubles make every production object under test look broken (the "my finger hurts" joke). Example (Java): write `FactoryOfPhrasesTest` for the fake that builds `Phrases`.

- **Atq.27 Never log inside a unit test** -> add to §7: Replace `Logger.debug(...)` in tests with a real matcher assertion; console output is not a test. Why: logging transfers knowledge to the author's head only, hides weak assertions, and produces noise for future developers. Example (Java):
  ```java
  @Test
  void buildsSimpleXml() {
    assertThat(
      XhtmlMatchers.xhtml(new Foo().build()),
      XhtmlMatchers.hasXPath("//foo")
    );
  }
  ```

- **Atq.29 Give every assertion a descriptive reason** -> add to §7: Use `assertThat(reason, actual, matcher)` with a human-readable message. Why: when the build breaks, the message should point directly at the wrong expectation. Example (Java):
  ```java
  assertThat("Total count of greetings", p.greetings().count(), equalTo(1));
  ```

- **Atq.30 Distinguish fast and deep tests; tag them** -> add to §7: Tag tests `@Tag("fast")` (<20ms, mocked) or `@Tag("deep")` (integration, real resources); run fast locally, deep on CI. Why: deep tests are inherently slow but catch leaks/limits unit tests miss; mislabeling all tests "unit/integration" obscures the real trade-off. Example (Java):
  ```java
  @Test @Tag("fast")
  void readsSomeData() throws IOException { ... }

  @Test @Tag("deep")
  void readsFromManyFiles(@TempDir Path tmp) throws IOException { ... }
  ```

- **Atq.31 Prove thread-safety with a latch-driven parallel test** -> add to §7: Submit N callables that wait on one `CountDownLatch`, then assert all succeeded (or that overlaps occurred). Why: submitting sequentially often runs one-at-a-time and proves nothing; forcing overlap exposes races. Example (Java):
  ```java
  final CountDownLatch latch = new CountDownLatch(1);
  for (int t = 0; t < threads; ++t) {
    futures.add(service.submit(() -> {
      latch.await();
      return books.add(title);
    }));
  }
  latch.countDown();
  ```

- **Atq.32 Use Cactoos `Threads`/`RunsInThreads` for concurrency tests** -> add to §7: Express a thread-safety test as a `Func` matcher instead of hand-rolled executors. Why: it is compact and reuses the correct latch semantics. Example (Java):
  ```java
  MatcherAssert.assertThat(
    t -> {
      final String title = String.format("Book #%d", t.getAndIncrement());
      final int id = books.add(title);
      return books.title(id).equals(title);
    },
    new Threads<>(new AtomicInteger(), 10)
  );
  ```

- **Atq.33 Submit tests in a separate PR from the code** -> add to §7: First PR modifies/disabled tests (reviewers validate intent), second PR fixes code without touching tests. Why: it prevents "fixing the tests to pass" cheating and separates requirements from implementation. Example (Java):
  ```java
  @Disabled("will pass after the fix in the next PR")
  @Test void fibo23() { assertThat(fibo(23), equalTo(28657)); }
  ```

- **Atq.34 Send a disabled test with a puzzle instead of a prose bug report** -> add to §7: Contribute a `@Disabled`/`#[ignore]` test plus a `@todo #N` puzzle that explains the defect. Why: the PR doubles as bug report and reproduction, and the puzzle is auto-converted into a ticket. Example (Java):
  ```java
  // @todo #42 fibo(23) returns 17711 but should return 28657
  @Disabled("fibo() is broken for this input")
  @Test void calculates23rdFibonacci() {
    assertThat(fibo(23), equalTo(28657));
  }
  ```

- **Atq.35 No pull request without a supporting test** -> add to §7: Every contribution must include a test that covers the change. Why: tests are the warranty protecting the employer's investment — previously paid-for code keeps working under refactoring. Example: reject any PR that changes behavior but adds zero test methods.
  *(also Apb.16)*

- **Atq.36 Test-scaffolding prerequisites via JUnit5 extensions, or better, fakes** -> add to §7: Prefer parameter injection through a `ParameterResolver` extension, and ultimately fake objects, over `@Before`/static builders. Why: extensions decouple setup from test methods; fakes are portable and testable. Example (Java):
  ```java
  @ExtendWith(PhrasesExtension.class)
  final class PhrasesTest {
    @Test
    void countsSimpleGreetings(Phrases p) {
      assertThat(p.greetings().count(), equalTo(1));
    }
  }
  ```

- **Atq.37 Long test classes are acceptable** -> add to §7: Do not split a big `FooTest` just for size; a test class is a container of scripts, not an object. Why: one-test-class-per-live-class is more valuable than line-count aesthetics. Example: a 5000-line `PhrasesTest.java` that maps to `Phrases.java` is fine.

- **Atq.38 Test HTTP clients against a real mock server** -> add to §7: Use `jcabi-http`'s `MkGrizzlyContainer` to serve queued answers and later inspect received requests. Why: it exercises the real HTTP stack and doubles as `verify` without a mocking framework. Example (Java):
  ```java
  final MkContainer container = new MkGrizzlyContainer()
    .next(new MkAnswer.Simple("hello, world!"))
    .start();
  try {
    new JdkRequest(container.home()).fetch()
      .assertBody(Matchers.containsString("hello"));
  } finally {
    container.stop();
  }
  ```
  *(also Adt.26)*

- **Atq.39 More test code than production code is healthy** -> add to §7: Don't treat a high test-to-code ratio as a smell (1:2 or more is fine); higher ratio means higher confidence. Why: tests are part of the code, not a separate product, and extra coverage means fewer regressions. Example: `jcabi-github` runs ~1:2.3 production:test+fake.

- **Atq.40 Refactoring is a bug, not part of a test fix** -> add to §7: Never refactor while fixing a failing test; file and pay for the refactor separately. Why: unpaid, unrequested code changes violate project coordination and muddy the fix. Example: fix the production method to make the test green, then open a separate ticket "simplify class X".

- **Atq.41 Write tests after the code, driven by reported bugs** -> add to §7: Implement, deploy, let users/testers break it, then write a test that reproduces each reported bug before fixing. Why: you can't test what you can't yet see, and testing exactly the visible, business-tolerable bugs avoids wasting resources. Example: see the code→deploy→break→test→fix loop graph (tdx tool).

- **Atq.42 Assert only on values you actually care about** -> add to §7: Replace `assertThat(xml, notNullValue())` after a `toString()` call — which already NPEs — with a content matcher such as an XPath assertion. Why: a weak assertion gives false confidence; matchers like `XhtmlMatchers.hasXPath("//foo")` encode intent. Example (Java): see Atq.27.

- **Atq.43 Keep test literals distinct to surface real intent** -> add to §7: When a static analyzer flags many identical literals, use different values in different tests instead of hoisting a constant. Why: the analyzer is asking whether the repetition is meaningful; usually it is not. Example (Java):
  ```java
  @Test void simplyWorks() { final String msg = "something"; ... }
  @Test void simplyWorksAgain() { final String msg = "something else"; ... }
  ```

- **Adt.22 Assert with Hamcrest `assertThat`, not `Assert`** -> add to §7: Replace procedural JUnit assertions with object-oriented matchers, and use libraries like jcabi-matchers for XPath/XML. Why: matchers compose, are reusable, and read declaratively. Example:
  ```java
  MatcherAssert.assertThat(
    new Foo().createXml(),
    XhtmlMatchers.hasXPaths("/document[count(message)=1]", "/document/message[.='hello, world!']")
  );
  ```

- **Adt.23 Test integration stubs start/stop around the right Maven phases** -> add to §7: Start supplementary servers in `pre-integration-test`, run tests in `integration-test`, and stop them in `post-integration-test`; run tests against a *real* MySQL/DynamoDB rather than mocks. Why: the phased lifecycle is what makes integration tests hermetic and repeatable; test real servers to catch real SQL behavior.

- **Adt.25 Validate generated HTML/CSS in integration tests** -> add to §7: Run jcabi-w3c against every page produced, asserting `valid()` or inspecting `errors()`. Why: it prevents small markup defects from becoming larger rendering problems later. Example:
  ```java
  Collection<Defect> defects = ValidatorBuilder.html().validate(page).errors();
  ```

- **Ada.26 Deliver a working skeleton backed by tests, not infrastructure first** -> add to §7: The first increment shipped to the user is a runnable (even dummy) product with build automation and tests; do not ship the `Makefile`/tests as a release. Why: only the user perceives value from a product; a test harness with no product is pointless and unpaid. Example (Java):
  ```java
  // alpha: always prints 5, yet it runs and is tested
  public static void main(final String... argv) {
    System.out.println("5");
  }
  ```

- **Ada.27 Keep a test loop runnable from one command** -> add to §7: Wire a single command (`make`, `mvn test`) that builds and runs the tests so the build is usable as a development tool. Why: fast, one-command turnaround is what makes tests a habit rather than a chore. Example (Makefile):
  ```make
  all: wc test
  wc: wc.c
  	gcc -o wc wc.c
  test: wc
  	echo 'Hello, world! How are you?' | ./wc | grep '5'
  ```

- **Ada.28 Skinny designs are easier to unit-test by construction** -> add to §7: When parsing is done in a decorator taking raw text, test it by passing fake text to the constructor; do not run a container to test a single object. Why: mocking one interface is far simpler than mocking a hierarchy or standing up a database. Example (Java):
  ```java
  final String name = new TxtAuthor("Yegor <y@example.com>").name();
  // no PostgreSQL server required
  ```

- **Acs.23 Judge clarity by an outsider's ability to fix a bug fast** -> add to §7: Test maintainability operationally: give a stranger the code and ask them to fix a bug in under an hour; treat long onboarding as a defect in the code, not the reader. Why: "clean" (no anti-patterns) is not enough — "clear" means a newcomer can use and modify it without help; only letting strangers contribute reveals whether the code speaks their language. Example (Java):
  ```java
  // PASS only if an outsider, with no help, fixes a failing test in < 1 hour
  assertThat(new TextFile(file).grep(pattern), equalTo(3));
  ```

- **Apa.23 Start every task with a failing test that reproduces the problem** -> add to §7: Before fixing a bug or adding a feature, write the unit test that makes the build red. Why: catching the bug with a test is "more than 80% of success"; the fix becomes trivial once red is reproduced. Example:
  ```java
  @Test void savesDataIntoDatabase() {
    DataBridge mapper = mock(DataBridge.class);
    Writer writer = new DBWriter(mapper);
    writer.write("hello, world!");
    verify(mapper).insert("hello, world!");
  }
  ```
  *(also Apa.25, Atq.24, Apb.14, Apb.15, Ada.25)*

- **Apa.24 If out of time, commit the failing test and stop** -> add to §7: Skip the test with `@Ignore`, add a `@todo` puzzle explaining what must be fixed, commit and close the task. Why: an isolated, documented failure is a complete deliverable and hands a precise next step to the team. Example:
  ```java
  @Ignore
  @Test void savesDataIntoDatabase() { /* @todo #123 reproduce+fix when... */ }
  ```

- **Apa.26 Measure coverage on every build and fail below the threshold** -> add to §7: Collect coverage in CI; if a new class drops coverage below the pre-set bar (~75–80%), break the build. Why: unknown coverage is worse than low coverage; the gate makes test writing non-optional. Example:
  ```text
  Threshold = 75% (or 80%); new class without unit test => coverage drops => BUILD FAILS
  ```

- **Apa.27 Prove a bug's absence when you cannot reproduce it** -> add to §7: Add a passing test that asserts the code works as intended; report resolved with that test as evidence. Why: repeated cycles of recording reality eventually let someone catch the real defect. Example:
  ```text
  // "Our software works correctly; here is the proof: see the new unit test."
  ```

- **Apb.13 Automated tests are the project's safety net** -> add to §7: Write the build pipeline and a
  first set of automated tests *before* writing feature code; they must run locally on every change and
  automatically before any merge to trunk. Why: a net of regression tests lets you refactor and add
  features faster because you know you cannot break yesterday's work; coding without it is like working
  at height without a net. Example:
  ```text
  day 1: build pipeline + a few unit tests
  day 2..n: feature code protected by the net
  ```

### Additions from real codebases

- **The canonical stack is JUnit 5 + Hamcrest, single-statement tests, no fixtures.** Cactoos:
  `junit-jupiter 6.1.3`, 1627 `@Test`, **0 `@BeforeEach`/`@BeforeAll`** (setup is composition inside
  the assertion), `assertThrows` **0** (exceptions asserted with OO matchers), Mockito 1 hit. Pattern
  (`scalar/AndTest.java:20-31`):
  ```java
  @Test
  void allTrue() {
      MatcherAssert.assertThat(
          "Each object must be True",
          new And(new True(), new True(), new True()),
          new HasValue<>(true)
      );
  }
  ```
- **OO matchers are enforced, not just preferred.** `cactoos-matchers` (`HasValue`, `IsText`, …);
  `forbidden-apis.txt:1-3` *bans* `org.hamcrest.Matchers` and `org.junit.jupiter.api.Assertions` in the
  packages already migrated, with `@todo #1434` tracking the ratchet (`pom.xml:404-412`).
- **Data-driven tests only when the matrix is genuinely shared.** Cactoos `NumberOfScalarsTest
  implements ArgumentsProvider` feeding `@MethodSource` (22 `@ParameterizedTest`) —
  `number/NumberOfScalarsTest.java:22-25,136-145`.
- **Takes: fast/deep split and real-server tests.** 640 `@Test`, 42 `@Tag("deep")`, 178/207 files use
  `assertThat`; a `Take` is driven in-process with `RqFake`/`RsPrint` (`facets/fork/TkForkTest.java:22-40`);
  integration tests bind a real server on `new ServerSocket(0)` via `FtRemote`
  (`http/FtRemote.java:67`) and via `maven-invoker-plugin` (`src/it/file-manager`). `deep`/`performance`
  profiles select slow tests; surefire excludes `deep` by default (`pom.xml:81,729-742`).
- **eo: parallelism, flakiness control, test linting.** JUnit 5 parallel execution via
  `configurationParameters`; `runOrder=random` in ITs; `rerunner-jupiter` retries flaky ITs; nightly
  `test-repetition.sh --max 10`; `jtcop-maven-plugin 1.4.4` lints test code (`ignoreGeneratedTests`);
  ArchUnit for architecture; JMH micro-benchmarks; Mockito ≈0.
- **Fakes are first-class production objects.** s3auth ships `Mk*` mocks and fluent `*Mocker` builders
  in `src/main` (`HostMocker`, `ResourceMocker`, `BucketMocker`, …); rehttp ships `FakeBase`/`FakeStatus`
  in `src/main` (used by tests and demos); rultor uses `new MkGitHub()` / `Profile.Fixed()`; jare ships
  a whole `io.jare.fake.Fk*` package.
- **Property-based tests and cross-implementation ITs.** xembly uses jqwik
  (`XemblerTest.java:165-166`) and runs the same suite against Saxon and Xerces (`src/it/saxon`,
  `src/it/xerces`); s3auth/rehttp spin up real dependencies for ITs.
- **Mutation coverage, not just line coverage.** Cactoos PIT `mutationThreshold=75` (`pom.xml:284`);
  xembly/`code-s3auth` PIT `mutationThreshold 80`; eo's per-module Jacoco gates are multi-counter
  (INSTRUCTION/LINE/BRANCH/COMPLEXITY/METHOD + `CLASS missed`).
- **Guard optional tooling with `Assumptions`, never delete the test.** rehttp guards XSL resources
  (`Assumptions.assumeFalse`, `TkAppTest.java:35-40`) and an external `xsltproc` binary
  (`CompilerTest.java:154-168`). Note: `@Tag` count is **0** in requs and rehttp — do not claim these
  repos tag fast/deep.

## 8. Static analysis, quality and style

- **B8.1 Measure class size by public methods, then by LOC** — A class must have fewer than five
  public/protected methods and stay under 250 LOC (Java; 100 in Ruby). Why: these two metrics capture
  focus (cohesion) and length (comprehensibility), the two drivers of maintainability.
  Example (Java):
  ```java
  final class WebPage {                       // 2 public methods, small
      private final URI uri;
      WebPage(URI path) { this.uri = path; }
      String content() { /* ... */ return ""; }
      void update(String content) { /* ... */ }
  }
  ```

- **B8.2 Maintainability is the prime quality metric** — Define code quality by the time it takes a
  newcomer to understand it; optimize for that above performance. Why: "if I don't understand you,
  it's your fault"; maintainability determines cost and lifecycle of software.
  Example (Java):
  ```java
  // Prefer the readable object form over a terse imperative optimization.
  Iterable<Integer> evens = new Filtered<>(numbers, number -> number % 2 == 0);
  ```

- **B8.3 Simple code beats clever code** — Assume the reader is a junior; write simple,
  self-explanatory code. Why: bad programmers write complex code; good programmers write simple code.
  Example (Java):
  ```java
  Employee jeff = department.employee("Jeff");
  jeff.giveRaise(new Cash("$5,000"));
  if (jeff.performance() < 3.5) { jeff.fire(); }
  ```

- **B8.4 No comments or Javadoc for internals; make names carry meaning** — Ideal code explains
  itself; document only external interfaces. Why: if a class needs documentation, its design is
  unclear; unit tests are the better documentation.
  Example (Java):
  ```java
  // wrong: "know-it-all" code needing comments
  class Helper { int saveAndCheck(float x) { return 0; } float extract(String text) { return 0; } }
  // right: self-documenting
  class WebPage { String content() { return ""; } void update(String content) { } }
  ```

- **B8.5 Keep cohesion: every method should use all properties** — If some properties are used by
  only some methods, the class has unrelated parts and low cohesion. Why: cohesion keeps a class
  focused on one responsibility; low cohesion signals a class that should be split.
  Example (Java):
  ```java
  // A class whose methods each touch all fields is cohesive; if not, extract sub-objects.
  ```

- **B8.6 Prefer declarative style to imperative style everywhere** — Write declarations, not
  algorithms; use component objects (`Filtered`, `Sorted`, `If`). Why: declarative code is more
  optimal (lazy), more polymorphic (decoupled), more expressive and free of temporal coupling.
  Example (Java):
  ```java
  Collection<Integer> evens = new Filtered<>(numbers, number -> number % 2 == 0);
  ```

- **B8.7 Remove code duplication with micro-classes, never with shared constants** — If two classes
  repeat a fragment, extract a class that encapsulates the *functionality*. Why: a shared constant
  couples classes and destroys cohesion; a class keeps semantics encapsulated and behavior
  replaceable.
  Example (Java):
  ```java
  // wrong: shared dumb literal
  class Constants { public static final String CRLF = "\r\n"; }
  // right: shared behavior in a class
  class CRLFString { private final String origin; CRLFString(String s) { this.origin = s; } @Override public String toString() { return String.format("%s\r\n", origin); } }
  ```

- **B8.8 Do not combine imperative and declarative styles** — Once statics/imperatives appear,
  declarative code can no longer use ctors and encapsulation properly, and the whole codebase drifts
  imperative. Why: the two styles are technically incompatible; mixing dooms the declarative portion.
  Example (Java):
  ```java
  // wrong: declarative class forced to call statics
  final class Between { private final Number num; Between(int l, int r, int x) { this.num = new Min(new Max(l, x), r); } }
  ```

- **B8.9 Quality is enforced by discipline, not motivation** — Adopt the EO rules as absolutes
  ("no exceptions", "never", "period"). Why: consistency (uniformity) is what makes an entire codebase
  predictable and refactorable.
  Example (Java):
  ```java
  // Uniform ctor discipline: primary ctor always last, always code-free.
  ```

- **B8.10 Keep tests and production under the same size/style limits** — Apply the 250-LOC and
  five-method limits to test code too. Why: if tests are held to a lower standard they become the
  least maintainable part of the project and get abandoned.
  Example (Java):
  ```java
  // Small focused test classes, one assertThat per test, fakes instead of mocks.
  ```

- **B8.11 Prefer final/abstract declarations as a static invariant** — The compiler should enforce
  the design: `final` fields, `final`/`abstract` classes, `final` methods in abstract classes. Why:
  machine-checkable rules prevent accidental violations and document intent.
  Example (Java):
  ```java
  abstract class Document { public abstract byte[] content(); public final int length() { return content().length; } }
  ```

*Note:* Volume 1 does NOT prescribe a named static-analysis toolchain (Qulice/Checkstyle/PMD/
FindBugs/Jacoco coverage gates). Those come from other project sources (see `EO-BRIEF.md`); they are
not grounded in this OCR.


### Additions from articles

- **Acoa.33 Never declare public static literals — encapsulate the data in a class** -> add to §8:
  Replace `CharEncoding.UTF_8` usage with `new UTF8String(array)`; a shared constant doesn't remove
  duplication, it removes only the *data* copy while encouraging duplicated *functionality*. Why:
  public static literals are unbreakable hard-coded dependencies (same sin as utility classes).
  Example (Java):
  ```java
  // bad: new String(array, CharEncoding.UTF_8);
  String text = new UTF8String(array).toString(); // good: behavior encapsulated
  ```

- **Acoa.34 Treat any static method as a smell and rewrite it to an object** -> add to §8: Static
  methods implement class behavior, not object behavior, are global variables, are impossible to
  mock, and are not thread-safe by definition. Why: they defeat scope decomposition. Example
  (bad): `class File { public static int size(String file) {...} }`.

- **Acoa.35 Avoid multiple `return`s and imperative operators as a style rule** -> add to §8: Until
  Java has `If`/`GreaterThan` objects, enforce a single `return` per method to at least resemble pure
  OOP. Why: `if/else` + two `return`s are procedural constructs. Example (Java):
  ```java
  public int max(int a, int b) { return new If(new GreaterThan(a, b), a, b); }
  ```

- **Acoa.36 Remove `instanceof` and class casting; use overloading or polymorphism** -> add to §8:
  `sizeOf(Iterable)` that special-cases `Collection` hides a coupling and grows an if/then fork; split
  into overloads (`sizeOf(Iterable)`, `sizeOf(Collection)`) or distinct doors. Why: casting
  "discriminates" against objects that already honor the contract. Example (Java):
  ```java
  int sizeOf(Iterable items) { int size = 0; for (Object i : items) { ++size; } return size; }
  int sizeOf(Collection items) { return items.size(); }
  ```

- **Acoa.37 Break temporal coupling by chaining immutable results instead of sequencing statements** ->
  add to §8: `Foo.with(Foo.with(new LinkedList(), "Jeff"), "Walter")` replaces `append(list,...); append(list,...); return list;`, and `withEnoughSpace(list).add("Walter")` replaces a separate validation call. Why: ordered statements carry hidden knowledge the compiler can't protect; a single expression removes the order. Example (Java):
  ```java
  list.add("Jeff");
  Foo.withEnoughSpace(list).add("Walter"); // validation cannot be forgotten/reordered
  ```

- **Acob.38 Do not provide auto-formatting; make contributors learn the rules** -> add to §8: when the build rejects a PR on style, let the author fix it manually rather than running a polisher. Why: Qulice encodes 900+ rules, some specifically EO; auto-formatting would hide the reasoning so contributors "never learn what the project wants and why". The author explicitly wants contributors to "suffer in order to learn".
  ```java
  final class Doc {
    private final File file;       // final class, final attr, this.-prefixed
    public void remove() { if (this.file.exists()) { this.file.delete(); } }
  }
  ```

- **Acob.39 Use concrete cohesion thresholds as refactoring triggers** -> add to §8: question cohesion when a class has more than seven methods or more than four attributes, or when any non-constructor method takes more than two arguments. Why: smaller, highly cohesive classes are empirically less error-prone (Basili et al.); the thresholds are practical heuristics for when to extract a new entity.
  ```java
  // Triggers: methods > 7, attributes > 4, method args > 2
  ```

- **Acob.40 Measure cohesion to prove EO claims** -> add to §8: use a cohesion calculator (jPeek) to compare class cohorts (e.g. with vs. without static methods, mutable vs. immutable, DTOs) rather than asserting EO principles dogmatically. Why: low cohesion = attributes/methods not related (splittable); high cohesion = nothing can be extracted. jPeek implements 12+ cohesion metrics and can turn EO claims into empirical results.
  ```java
  final class Books {            // low cohesion: titles and prices unrelated
    private List<String> titles;
    private List<Integer> prices;
  }
  ```

- **Acob.41 Measure the "distance of coupling" of returned data** -> add to §8: count how far a returned value is manipulated after it leaves an object; a high distance (e.g. `toString().split().parseInt()`) signals a Tell-Don't-Ask violation. Why: encapsulation is a discipline, not a barrier; the proposed static metric detects clients that deconstruct returned values, giving an objective coupling signal.
  ```java
  String[] parts = x.toString().split(" ");   // distance 2: split, then parseInt
  int t = Integer.parseInt(parts[0]);
  ```

- **Acob.42 Define quality as defects found before users: Q = F / (F + U)** -> add to §8: track bugs found internally (F) vs. reported by users (U); improve quality by increasing F, not by preventing bugs. Why: total bugs are infinite, so the only lever is finding more before release; 100% means no user-visible bugs (test escapes), 0% means users find them all.
  ```text
  quality = F / (F + U)     // F = found by team, U = found by users
  ```

- **Atq.44 Make Qulice (Checkstyle+PMD+FindBugs) mandatory and fail the build** -> add to §8: Run an aggregated static analyzer in maximum-strict mode; reject any branch that violates even one rule. Why: ~900 checks turn a shared industry standard into unbreakable project policy and remove architect discretion. Example: add the Qulice Maven plugin as a build step; a violation rejects the merge.
  *(also Adt.27, Apa.28, Apb.17, Apb.18, Acs.31)*

- **Atq.45 Finalize public methods** -> add to §8: Every non-abstract public method must be `final` so subclasses cannot override and break the superclass. Why: predictability of design; a method that is never overridable is also never a surprise. Example (Java):
  ```java
  final class Employee {
    public final String name() { return "Jeff"; }
  }
  ```

- **Atq.46 Treat inconsistent style and missing docs as bugs** -> add to §8: File defects for inconsistent code style, incomplete/absent documentation, over-complex code, and a missing style guide. Why: maintainability is a non-functional requirement, and modern software value is largely maintainability. Example: "class `Foo` has no Javadoc explaining usage" is a bug, not a task.
  *(also Atq.49)*

- **Atq.47 Prioritize maintainability over functionality under constraints** -> add to §8: When time/budget force a trade-off, keep the design clean and let functionality be incomplete/buggy; functional bugs should be the cheap ones. Why: unmaintainable code must eventually be rewritten, while a missing feature can be added later. Example: prefer a correctly factored `Text`/`Words` split over a monolithic `FileUtils.readWords` that does everything.

- **Atq.48 Fewer language options track higher quality** -> add to §8: Prefer a strict, restricted style and forbid syntactic sugar; the more ways there are to express the same thing, the lower the maintainability. Why: restrictions reduce creative variance and make code easier to read. Example: use consistent `if (x) { ... }` blocks and rely on Checkstyle to ban alternatives.

- **Atq.50 Enforce both coverage and mutation thresholds** -> add to §8: Configure the build to fail below a high test-coverage bar and a mutation-coverage bar. Why: coverage alone can be gamed by assertion-free tests; mutation coverage proves the tests detect real changes. Example: gate the pipeline on Jacoco coverage plus a mutation threshold (as in EO projects).
  *(also Adt.28)*

- **Atq.51 Keep the repository small to keep quality high** -> add to §8: Extract reusable pieces into standalone small repositories rather than one monorepo. Why: small repos permit maximum lint strictness, deeper tests without slow builds, pedantic reviews, a complete README, frequent releases, and effective AI-agent context. Example: publish `cactoos`-style focused libraries instead of a giant enterprise repo.

- **Atq.52 Make the quality wall impossible to go around** -> add to §8: Enforce quality through automated pre-flight builds, a read-only `master`, high coverage, mandatory static analysis, and multi-step reviews — not through programmer goodwill. Why: quality exists only if controlled and enforced; delegated quality control leads to pressure and compromise. Example: see Atq.44/Atq.50 and a pre-flight branch build.

- **Adt.29 Limit interface method count; treat overloads as smelly** -> add to §8: Flag interfaces with more than three methods as refactoring candidates and overloaded interface methods as serious trouble; extract a "smart" decorator instead. Why: small interfaces are the mechanical expression of ISP and keep types cohesive.

- **Adt.30 Avoid one-time variables and redundant constants/imports** -> add to §8: Inline a variable used only once (`new File("data.txt")`), and don't create constants for arbitrary literals — only for real-world characteristics. Why: such code pollutes the namespace and adds no meaning; Qulice can catch some of this, but not all.
  *(also Acs.29)*

- **Adt.31 Follow paired-bracket indentation, 80-column wrapping** -> add to §8: A bracket either ends a line or closes on the same line; put as much as possible on one line within 80 chars. Why: it produces a consistent, machine-checkable style that keeps diffs small.

- **Ada.29 Keep one language/technology per repository for a single style** -> add to §8: Prefer a repo that uses a single language or technology so a single coding standard can be enforced across it. Why: a mixed repo develops divergent styles that are almost impossible to unify. Example (text):
  ```text
  colorizejs repo: colorize.js + test-colorize.js — one language, one standard.
  ```

- **Ada.30 Keep metrics accurate by decomposing before measuring** -> add to §8: Compute lines-of-code, cohesion and coupling per small repo, never across a mixed mega-repo. Why: a single metric on a repo holding 200k Java + 150k XML + 50k JS tells you nothing and cannot be compared with other projects. Example (text):
  ```text
  Prefer: one repo = one language = comparable, meaningful metrics.
  ```

- **Acs.24 Prohibit empty lines inside method bodies** -> add to §8: Ban blank lines within methods (and enforce it in the build); when you feel the urge to separate parts, extract a private method or a new class. Why: an empty line means a method has "parts" and does more than one thing; functional decomposition belongs in language constructs, not in whitespace, and emptiness invites multi-page methods. Example (Java):
  ```java
  public int grep(Pattern regex) throws IOException {
    return this.count(this.lines(), regex);   // no blank line: two things became two methods
  }
  ```

- **Acs.25 Obey Paired Brackets** -> add to §8: A bracket must either start/end a line or be paired on the same line; a closing bracket starts at the same indentation as its opening line. Why: consistent paired placement produces vertical guide lines that make nesting and boundaries instantly visible in any language and in JSON/arrays/fluent chains. Example (Java):
  ```java
  new Foo(
    Math.max(10, 40),
    String.format(
      "hello, %s",
      new Name(
        Arrays.asList("Jeff", "Lebowski")
      )
    )
  );
  ```

- **Acs.26 Enforce monotonic indentation** -> add to §8: Adjacent lines may increase indentation by exactly one unit; they may decrease by any amount; lines never sit between grid steps. Why: it settles formatting disputes with a mechanical rule, and together with Paired Brackets it makes brackets line up vertically and indentation never jump further than necessary. Example (Java):
  ```java
  class Repository {
  →→void save(File file,
    →→String text, String summary,
      boolean overwrite) {
      if (file.exists()
      →→&& !overwrite) {
        throw new IllegalStateException(
        →→"file already exists");
      }
    }
  }
  ```

- **Acs.27 Prohibit documentation comments** -> add to §8: Do not write Javadoc or inline comments; make names and structure self-explanatory and let tooling (an LLM) explain intent on demand, failing the build when code cannot be interpreted. Why: comments are unclear to future readers and decay into lies that cause bugs; they introduce a second source of truth that contradicts the code, so an automatic Code Interpretability Score gate is preferable. Example (Java):
  ```java
  int shortest(int[][] g, int a, int b) {
    // no Javadoc block — the method name and code must speak for themselves
    return this.path(g, a, b).length();
  }
  ```

- **Acs.28 Optimize for clarity (a stranger's speed), not just cleanliness** -> add to §8: Beyond removing code smells, make code understandable to outsiders; keep it open and invite bug reports about anything unclear. Why: a clean-but-cryptic class is as useless as a kitchen whose panel speaks French; maintainability is defined by how fast a stranger can modify and fix it. Example (Java):
  ```java
  // Prefer a short, intention-revealing API over clever but opaque field/algorithm names
  final int matches = new TextFile(file).grep(pattern);
  ```

- **Acs.30 Kill if-then-else forks that can become decorators** -> add to §8: When an `if/else` decides whether or how to run a method, move it to a decorator; inline forking is the smell. Why: the branching condition does not belong to the object doing the work; a decorator keeps the core small and lets behavior be composed rather than hard-coded. Example (Java):
  ```java
  final class QuickTalk implements Talk {
    private final Talk origin;
    QuickTalk(Talk t) { this.origin = t; }
    void modify(Collection<Directive> dirs) {
      if (!dirs.isEmpty()) { this.origin.modify(dirs); }
    }
  }
  ```

- **Apa.29 Make the CI pipeline as fragile and strict as possible** -> add to §8: The architect's goal is a "guard wall" where any reproducible minor error fails the build predictably. Why: fragility is a success factor; chaos enters through pull requests from programmers who care less about overall quality. Example:
  ```text
  Pipeline must fail on: formatting, static rule, unit/integration test, coverage drop,
  multi-platform build break, doc-generation break.
  ```

- **Apa.30 Eliminate the seven maintainability sins** -> add to §8: Hunt anti-patterns, untraceable changes, ad-hoc releases, volunteer static analysis, unknown coverage, nonstop development and undocumented interfaces. Why: these are the fatal, self-reinforcing causes of unmaintainable code. Example:
  ```text
  1 anti-patterns  2 untraceable changes  3 ad-hoc releases  4 volunteer static analysis
  5 unknown coverage  6 nonstop development (no releases)  7 undocumented interfaces
  ```

- **Apa.31 Document external interfaces, not internals** -> add to §8: Write user-facing docs (README, usage) but keep documentation comments out of source internals. Why: a newcomer learns the product through its interfaces; working software beats comprehensive internal docs. Example:
  ```text
  README.md = external interface + usage; source = self-documenting names, no prose comments
  ```

- **Apb.19 Carrot-and-stick for AI agents** -> add to §8: To make an AI coding agent follow house
  style, give it an inspirational manifesto file (`CLAUDE.md`/skill prompt) plus hard static checkers
  that punish violations; formalize half of "beauty" as checkers. Why: agents learn "functionality
  first" from billions of mediocre samples, so weak checks alone fail; a manifesto sets intent and
  checkers enforce the formalizable half. Example:
  ```text
  carrot: CLAUDE.md / SKILL.md describing elegant-object rules
  stick:  Qulice + rubocop-elegant run in the build, failing on violation
  ```

- **Apb.20 Interview for intolerance to chaos** -> add to §8: Assess candidates by asking them to
  review a small, deliberately flawed class and find structural defects; prioritize the ones annoyed by
  inconsistency. Why: talent is "an innate need to structure things"; mediocre coders tolerate broken
  naming and missing contracts silently. Example:
  ```text
  test: give a class named `Parser` with get/save methods and inconsistency;
  ask the reviewer to list defects, most important (structural) first.
  ```

### Additions from real codebases

- **Qulice is the enforceable quality wall.** It bundles Checkstyle 14.1.0, PMD 7.26/7.28 and
  ErrorProne 2.50 behind one `verify`-phase goal (`CheckMojo.java:35-40`); runs validators in a
  5-thread executor and throws `MojoFailureException` on any `Violation` (`CheckMojo.java:187-191`);
  emits `validator: file[line]: message (RuleName)` (`CheckMojo.java:170-185`). Hard rules include:
  100-col lines, LF, no tabs/trailing spaces/double blank lines; `ReturnCount=1`; `ParameterNumber≤3`;
  statements ≤40; lambda body ≤20; final params/locals; no `++`/`--` expressions; no `clone`/`finalize`;
  `RequireThis`; full Javadoc with ordered, capitalised tags; unused suppressions are themselves
  violations (`ChecksTest`, `@checkstyle`, `UnknownSuppressionCheck`).
- **Analyzer contradictions are catalogued and resolved one rule at a time.** ~20 documented conflicts
  (Checkstyle `UnnecessaryParentheses` vs ErrorProne `OperatorPrecedence`; `ArrayTrailingComma` vs
  `NoArrayTrailingComma`; PMD `UseDiamondOperator` false positives; `CompareObjectsWithEquals` vs
  `UndefinedEquals`; `LawOfDemeter` vs fluent pipelines) — each disables exactly one rule with a
  written rationale, never a blanket `@SuppressWarnings` (Qulice `checks.xml`/`ruleset.xml`/`Xplugin.java`).
- **`forbiddenapis` is a ratchet.** Cactoos bans `org.hamcrest.Matchers`/`Assertions` only for an
  explicit `includes` list of already-converted packages, tracked by a PDD puzzle — ban it where you've
  won, grow the list, never regress (`pom.xml:396-432`).
- **Quality gates live in the POM, not a dashboard.** Cactoos Jacoco
  `INSTRUCTION ≥0.61, LINE ≥0.65, BRANCH ≥0.65, COMPLEXITY ≥0.57, METHOD ≥0.57, CLASS missed ≤15`
  (`pom.xml:331-369`) and PIT 75; `revapi` fails the build on breaking API changes and every accepted
  break carries its own `<justification>` (`pom.xml:137-257`); `maven-verifier-plugin` asserts
  `LICENSE.txt` contains `2017-2026`. eo uses per-module multi-counter Jacoco thresholds plus `jtcop`
  test linting and ArchUnit.
- **Style canon beyond the book.** full Javadoc is *required* by Qulice (contra "no comments"); 80/100
  column wrapping, paired brackets, no blank lines in method bodies (`Acs.24`), no static nested
  classes, `ParameterNumber≤3`, `ReturnCount=1`.
- **Commit/repo hygiene gates run as independent workflows.** `reuse`/`copyrights` (SPDX + REUSE),
  `typos`, `xcop` (XML), `yamllint`, `markdown-lint`, `actionlint`, `simian` (duplication), `ort`
  (dependency-license audit). Each workflow is small, single-purpose, `timeout-minutes: 15`, and often
  PR-path-filtered (eo `qulice.yml` on `**.java`/`**/pom.xml`).

## 9. Build, CI/CD and releases

*Not covered by this source.* Volume 1 of "Elegant Objects" contains no material on build tooling,
continuous integration, release automation, versioning, or deployment. The only adjacent quality/
discipline idea in the book is that automated unit tests should act as a safety net for refactoring
(see B7.11, B7.12) and that a fast-failing build is preferable to a permissive one (see B6.4).

- **B9.1 Make the build fail fast on broken behavior** — Grounded in the book's fail-fast philosophy:
  a build that surfaces problems early is better than one that hides them. Why: the sooner a problem
  is revealed, the sooner it is fixed; concealment compounds defects. *(EO is not a CI guide, but its
  fail-fast rule applies directly to pipeline gates.)*
  Example (Java):
  ```java
  // A failing test must fail the build rather than be skipped/ignored.
  @Test public void lengthThrowsWhenMissing() { /* expects IOException */ }
  ```

- **B9.2 Treat unit tests as a mandatory part of every class in the pipeline** — Because a unit test
  is part of the class, a class without tests is incomplete and should not be released. Why: tests are
  the documentation and the refactoring safety net; shipping untested classes forfeits both.
  Example (Java):
  ```java
  // Every production class ships a corresponding test class.
  ```


### Additions from articles

- **Acoa.38 Ship a Maven-Central JAR per release with tests + fakes in the same artifact** -> add to
  §9: Built-in fake classes are published inside the production JAR (not a test classifier) so users
  can test with them. Why: the fakes are part of the library's contract for its users. No snippet.

- **Acob.43 Use a transpile-to-Java build pipeline with a foreign-object catalog** -> add to §9: for EO projects, parse -> optimize -> discover -> pull -> resolve -> place -> mark -> transpile -> compile -> test -> unplace/unspile -> copy -> deploy -> push -> merge, driven by a Maven plugin and a CSV catalog of foreign objects. Why: transpiling XMIR to Java (rather than bytecode) yields readable/debuggable output, reuses the Java compiler's optimizations, and simplifies the compiler; sources ship inside the JAR (`EO-SOURCES/`) for reproducibility and the Mark step.
  ```xml
  <plugin>
    <groupId>org.eolang</groupId>
    <artifactId>eo-maven-plugin</artifactId>
    <executions><execution><goals>
      <goal>register</goal><goal>assemble</goal><goal>transpile</goal>
      <goal>copy</goal><goal>unplace</goal><goal>unspile</goal>
    </goals></execution></executions>
  </plugin>
  ```

- **Atq.53 Make the build fragile on purpose** -> add to §9: Put strict static analysis, coverage, and mutation gates directly in the pre-flight build so bad code can never reach `master`. Why: a high quality bar deliberately makes casual code changes hard. Example: Rultor runs the build; any Qulice/coverage failure rejects the branch.

- **Atq.54 Run deep tests on the server, fast tests locally** -> add to §9: Configure Maven Surefire to run `@Tag("fast")` by default and `@Tag("slow"/"deep")` on CI (`mvn test -Dgroups=slow`). Why: programmers get a sub-second safety net; the server absorbs the slow integration tests. Example (XML):
  ```xml
  <plugin>
    <artifactId>maven-surefire-plugin</artifactId>
    <configuration><groups>fast</groups></configuration>
  </plugin>
  ```

- **Atq.55 Schedule integration tests against external dependencies** -> add to §9: Create integration tests for third-party APIs (e.g. a payment/blockchain API) and run the build on a daily CI schedule, emailing on red. Why: proactive risk mitigation detects a changed/removed API before users notice. Example: a daily cron job runs the API-contract ITCase and alerts the maintainer.

- **Atq.56 Separate unit-test and integration-test phases** -> add to §9: Use `Test` suffix for unit tests (fail fast at `test`) and `ITCase` for integration tests run across the four Maven phases `pre-integration-test → integration-test → post-integration-test → verify`. Why: resources (e.g. a MySQL instance) are acquired/released around tests, and failures are verified only at `verify`. Example: `GreetingsITCase.java` under `foo/it/`.

- **Atq.58 Tests and code in separate builds/PRs** -> add to §9: Because tests are merged before their implementation, the pipeline must tolerate temporarily disabled tests without lowering the coverage gate globally. Why: this keeps requirements review separate from implementation review. Example: `@Disabled` tests are still counted in review but excluded from the red signal until enabled.

- **Adt.33 Run every build in its own disposable Docker container** -> add to §9: Execute test/merge/release/deploy scripts inside a fresh container (`docker run --rm ... <cmd>`), configured via `.rultor.yml` `docker.image`. Why: isolation makes errors reproducible, packages installable in a clean OS, and the container is destroyed afterward. Example:
  ```bash
  sudo docker run --rm -i -t yegor256/rultor mvn clean test
  ```
  ```yaml
  docker:
    image: yegor256/beta
  ```

- **Adt.34 Run as a non-root user inside the container** -> add to §9: Because Docker starts as root, entry scripts should create a user, grant sudo, and `su` into the build script. Why: some tools (e.g. PostgreSQL `initdb`) refuse to run as root. Example:
  ```bash
  #!/usr/bin/env bash
  adduser --disabled-password --gecos '' r
  adduser r sudo
  echo '%sudo ALL=(ALL) NOPASSWD:ALL' >> /etc/sudoers
  su -m r -c /home/r/script.sh
  ```

- **Adt.36 Use `versions-maven-plugin` to set the release version** -> add to §9: In release scripts, set the project version from the tag with `mvn -ntp versions:set -DnewVersion=${tag}` (with `generateBackupPoms=false`) instead of editing POMs by hand. Why: one deterministic command rewrites all modules; no stale backup POMs.

- **Adt.37 Deploy to Maven Central through Sonatype OSSRH with signing** -> add to §9: Configure `distributionManagement` to `oss.sonatype.org` staging, the `maven-gpg-plugin` sign goal at `verify`, `maven-source-plugin`/`maven-javadoc-plugin`, and `nexus-staging-maven-plugin` with `deploy`+`release`. Why: Central requires sources, javadoc, and GPG signatures; the staging plugin releases the whole bundle atomically. Example:
  ```xml
  <plugin>
    <groupId>org.sonatype.plugins</groupId>
    <artifactId>nexus-staging-maven-plugin</artifactId>
    <extensions>true</extensions>
    <configuration>
      <serverId>oss.sonatype.org</serverId>
      <nexusUrl>https://oss.sonatype.org/</nexusUrl>
    </configuration>
  </plugin>
  ```

- **Adt.38 Encrypt secrets, never commit them in plaintext** -> add to §9: Keep GPG keyrings, `settings.xml`, RubyGems tokens, SSH keys encrypted (`rultor encrypt -p owner/repo file`) as `*.asc`, declare them under `decrypt:` in `.rultor.yml`, and decrypt only on the CI server. Why: credentials must never sit in the open even for collaborators; chat-driven deploys need on-the-fly GPG decryption. Example:
  ```yaml
  decrypt:
    settings.xml: "repo/settings.xml.asc"
    pubring.gpg: "repo/pubring.gpg.asc"
    secring.gpg: "repo/secring.gpg.asc"
  ```
  *(also Acs.33, Apb.28)*

- **Adt.39 Reserve random TCP ports; never hardcode them** -> add to §9: Use `build-helper-maven-plugin:reserve-network-port` and pass `${...port}` to the server under test. Why: hardcoded ports collide with other services and prevent parallel builds in CI. Example:
  ```xml
  <plugin>
    <groupId>org.codehaus.mojo</groupId>
    <artifactId>build-helper-maven-plugin</artifactId>
    <executions><execution><goals><goal>reserve-network-port</goal></goals>
      <configuration><portNames><portName>tomcat.port</portName></portNames></configuration>
    </execution></executions>
  </plugin>
  ```

- **Adt.40 Use maven-failsafe for integration tests; verify at the end** -> add to §9: Run ITs with `maven-failsafe-plugin` `integration-test` + `verify`, letting stubs start/stop around them, and defer build failure to `verify`. Why: failsafe records failures and lets `post-integration-test` teardown run before failing the build, unlike surefire.

- **Adt.41 Wrapper/bootstrap test dependencies as Maven plugins** -> add to §9: Download and unpack required binaries (DynamoDB Local, MySQL distribution, PhantomJS, Nutch) via `maven-dependency-plugin`/`download-maven-plugin`/`exec-maven-plugin` into `target/` during the correct phase. Why: it makes integration tests self-contained and reproducible on a clean machine. Example:
  ```xml
  <plugin>
    <groupId>com.github.klieber</groupId>
    <artifactId>phantomjs-maven-plugin</artifactId>
    <configuration><version>1.9.2</version></configuration>
  </plugin>
  ```

- **Adt.42 Split builds by purpose and cost: fast / cheap / preflight / proper** -> add to §9: Keep a few-seconds local `fast` build (unit tests + coverage), a <10-minute `cheap` CI build (integration + style), an up-to-an-hour `preflight` merge build (mutation/load/security), and a `proper` release build (multi-browser, A/B, regression). Why: one build length cannot fit both editing rhythm and release quality; never reduce to "one build fits all".

- **Adt.43 Use AppVeyor to continuously integrate Windows builds** -> add to §9: Add `appveyor.yml` to build Maven projects on Windows, caching Maven and `~/.m2`, and let Rultor wait for AppVeyor success before merging. Why: Java/Ruby builds that pass on Linux often fail on Windows; multi-platform CI catches that before merge. Example:
  ```yaml
  version: '{build}'
  os: Windows Server 2012
  install:
    - cmd: SET PATH=C:\maven\apache-maven-3.2.5\bin;%JAVA_HOME%\bin;%PATH%
    - cmd: SET MAVEN_OPTS=-XX:MaxPermSize=2g -Xmx4g
  build_script:
    - mvn clean package --batch-mode -DskipTest
  test_script:
    - mvn clean install --batch-mode
  cache:
    - C:\maven\
    - C:\Users\appveyor\.m2
  ```

- **Adt.44 Climb the CI maturity ladder deliberately** -> add to §9: Progress through: source code → one-line automated build → Git → pull requests → mandatory code review → tests → static analysis → pre-flight build → production-simulation container → stress tests. Why: each level names a concrete capability the pipeline must have; the build is red if any gate fails.

- **Adt.46 Host a private Maven repo on S3 with the S3 wagon** -> add to §9: Point `distributionManagement`/`repositories` at `s3://bucket/{release,snapshot}`, store IAM keys in `settings.xml`, and add the `maven-s3-wagon` build extension. Why: private artifacts need a repository that is not public Maven Central; S3 is cheap and Rultor can deploy to it automatically. Example:
  ```xml
  <extension>
    <groupId>org.kuali.maven.wagons</groupId>
    <artifactId>maven-s3-wagon</artifactId>
    <version>1.2.1</version>
  </extension>
  ```

- **Adt.47 Version database schema with Liquibase changesets** -> add to §9: Add `liquibase-maven-plugin`, a `master.xml` with `<includeAll>`, and numbered changeset files (`002-add-user-address.xml`) run per profile (`mvn liquibase:update -Pproduction`). Why: DB migrations must be ordered, versioned, and applied as part of the build; numeric prefixes guarantee chronological order.

- **Adt.48 Keep DB credentials and hosts in Maven profiles, not POMs** -> add to §9: Put `mysql.host/port/db` in `settings.xml` profiles (`-Pproduction`/`-Ptest`) and reference them from Liquibase/jdbc config. Why: keeps secrets and environment-specific values out of version control and switches environments with one flag.

- **Adt.49 Deploy to PaaS by pushing to its git remote** -> add to §9: For Heroku, add the remote, place the key in `~/.ssh`, and `git push -f heroku <branch>:master` from the release script. Why: it reuses the standard git deploy path and keeps deployment one-command and scripted. Example:
  ```bash
  git remote add heroku git@heroku.com:aintshy.git
  mv ../id_rsa ../id_rsa.pub ~/.ssh
  git push -f heroku $(git symbolic-ref --short HEAD):master
  ```

- **Adt.50 Publish to RubyGems via an encrypted token and a script** -> add to §9: `sed` the version from the tag into the gemspec, `gem build`, `chmod 0600`, then `gem push --config-file <decrypted rubygems.yml>`. Why: same one-command, credentials-never-committed pattern for non-Java artifacts. Example:
  ```yaml
  release:
    script: |
      rm -rf *.gem
      sed -i "s/1.0.snapshot/${tag}/g" foo.gemspec
      gem build foo.gemspec
      chmod 0600 /home/r/rubygems.yml
      gem push *.gem --config-file /home/r/rubygems.yml
  ```

- **Adt.51 Compile SASS and minify assets during the build** -> add to §9: Run `sass-maven-plugin` at `generate-resources`, minify CSS, then pick the result up via `maven-war-plugin` webResources. Why: stylesheet generation and minification are build steps; the WAR must contain only generated assets.

- **Adt.52 Verify pre-conditions on another CI before merging** -> add to §9: In the merge script, trigger AppVeyor for the PR, poll its status until `success`/`failed`, and fail the merge on failure. Why: a merge bot running only Linux/Docker cannot test Windows; delegate that check before merge. Example:
  ```bash
  ver=$(curl -K ../curl-appveyor.cfg --data "{accountName:'yegor256',projectSlug:'takes',pullRequestId:'${pull_id}'}" https://ci.appveyor.com/api/builds | jq -r '.version')
  while true; do
    status=$(curl -K ../curl-appveyor.cfg https://ci.appveyor.com/api/projects/yegor256/takes/build/${ver} | jq -r '.build.status')
    if [ "${status}" == "success" ]; then break; fi
    if [ "${status}" == "failed" ]; then exit 1; fi
    sleep 5s
  done
  ```

- **Adt.53 Run the whole app (or one take) in tests at a random port** -> add to §9: Start a test HTTP server via `FtRemote` and drive real requests with jcabi-http. Why: it exercises the deployable artifact over TCP without a fixed port, mirroring production more closely than unit stubs.

- **Adt.54 Simulate production in the pipeline** -> add to §9: At the top maturity levels, run the build in a container that simulates production environment and data, plus automated stress/performance tests every build. Why: production simulation and stress tests catch issues that unit/integration tests cannot.

- **Ada.31 Pin dependency versions by trust, not by dogma** -> add to §9: Use dynamic versions (`1.+`, `~>`) for libraries whose authors you trust to honor semantic versioning, and exact versions for untrusted ones. Why: fixed versions cause conflicts when another library needs a newer version, while dynamic versions plant a time bomb if the author breaks compatibility silently. Example (XML):
  ```xml
  <!-- trusted: dynamic -->
  <version>[1.13,2.0)</version>
  <!-- untrusted: fixed -->
  <version>1.13.5</version>
  ```

- **Ada.32 Small repos give fast builds and cheap CI** -> add to §9: Split the codebase so each repo's build finishes quickly (seconds, not minutes). Why: fast turnaround makes the build a usable development tool and keeps the CI pipeline cheap. Example (text):
  ```text
  colorizejs Travis build: 51 seconds — fast enough to iterate.
  ```

- **Ada.33 Release each component independently with its own pipeline** -> add to §9: A separately versioned component gets its own repo, build, README, license and release. Why: independent repos encapsulate one problem, enabling independent upgrade and reuse across projects. Example (text):
  ```text
  yegor256/colorizejs: package.json + README + LICENSE + CI + PDD + rultor.
  ```

- **Acs.32 Stamp version and Git hash into MANIFEST.MF** -> add to §9: At build time, inject the project version and the VCS hash as custom manifest attributes so the running artifact can report exactly what it is. Why: it lets the deployed WAR display its version and commit on every page, making support and debugging traceable to an immutable build. Example (Java):
  ```java
  String created = Manifests.read("Created-By");
  String version = Manifests.read("Foo-Version");
  String hash = Manifests.read("Foo-Hash");
  ```
  ```xml
  <manifestEntries>
    <Foo-Version>${project.version}</Foo-Version>
    <Foo-Hash>${buildNumber}</Foo-Hash>
  </manifestEntries>
  ```
  *(also Adt.45)*

- **Apa.32 Make master read-only; merge only through a bot/script** -> add to §9: Nobody pushes to `master`; every change lands via a verified merge bot (Rultor) that tests then merges. Why: raising the red flag *before* code enters master puts blame on the author and keeps the build clean. Example:
  ```text
  revoke write access to master; work through forks + PRs;
  @rultor merge => test => push; if any test fails the branch is rejected.
  ```
  *(also Atq.57, Adt.32, Apb.29)*

- **Apa.33 Do not rely on pre-flight branch builds alone** -> add to §9: A green branch build does not protect master (post-merge runs can break; a collaborator can still push directly). Why: only a merge gate on the write path guarantees a clean master. Example:
  ```text
  Pre-flight checks branches; merge bot checks the merged result before push.
  Both are needed; only the bot is on the critical path.
  ```

- **Apa.34 Automate the entire release to one command** -> add to §9: `./release.sh` (or a "please release now" ticket comment to a bot) must test, package, upload and deploy end-to-end. Why: ad-hoc, button-click releases are a classic unmaintainability sin and are unreproducible. Example:
  ```text
  $ ./release.sh
  ...
  DONE (took 98.7s)
  ```
  *(also Adt.35, Apb.25)*

- **Apa.35 Wire a real continuous-delivery pipeline, not just CI** -> add to §9: Chain build → package → upload artifact → build JavaDoc → deploy, all automated. Why: software is only a product when it is packagable and deployable in one click. Example:
  ```text
  CI: multi-platform build, unit + integration tests, static analysis, coverage, docs.
  CD: package JAR -> upload to repository -> build JavaDoc -> deploy to S3.
  ```

- **Apa.36 Version, tag and publish every release; keep every binary downloadable** -> add to §9: Use semantic versioning, git tags, release notes, and make every version downloadable forever. Why: a release history lets anyone understand where the project was and where it is going. Example:
  ```text
  v0.1.3 must still be downloadable even when the project works on 3.4;
  each release has tagged notes.
  ```
  *(also Apb.27)*

- **Apa.37 Prefer several small releases per day over rare big ones** -> add to §9: Re-deploy after every merged bug fix; keep increments tiny (30–60 min of work). Why: continuous small increments keep changes reviewable, revertible and always deployable. Example:
  ```text
  Increment = bug fix | bug report | feature | micro-step;
  release multiple times a day to champions.
  ```
  *(also Apb.24)*

- **Apa.38 Automate merging with a bot that refuses any failing branch** -> add to §9: The merge bot runs the full test suite and rejects branches that break even one test; the author must fix and retry. Why: it shifts responsibility to the author and prevents broken code from ever entering master. Example:
  ```text
  @rultor merge → merge + test → push master OR reject with the failing test
  ```

- **Apb.21 Never merge into a broken master** -> add to §9: Before making any change, check that ALL CI
  jobs (not just the Maven one) are green; if any is red, stop, report a bug, and wait until the team
  fixes it. Why: piling changes onto a broken build multiplies the cost of cleaning it up, so while the
  build is broken only build-fixing changes are accepted. Example:
  ```text
  red CI -> submit bug "master is broken" -> do not start your branch -> wait
  ```

- **Apb.22 Fix the build in a separate PR** -> add to §9: If you must repair the build yourself, submit
  the fix as its own pull request, never mixed with feature changes. Why: a build-fix diff must be
  reviewable and revertible in isolation; mixing it obscures what actually broke. Example:
  ```text
  PR A: "[build] fix flaky integration job"
  PR B: (only after A merges) your feature change
  ```

- **Apb.23 Strict pipeline: tests, static analysis, coverage** -> add to §9: No prototype is approved
  unless its build pipeline includes (1) unit testing, (2) static analysis, and (3) test-coverage
  control; these three are absolutely mandatory. Why: this trio is the minimum that enforces quality
  automatically and protects the team from a broken baseline. Example:
  ```text
  .github/workflows/build.yml -> junit + qulice + jacoco (fail under threshold)
  ```

- **Apb.26 Badges show the pipeline is watched** -> add to §9: Add CI/build/coverage badges to the repo
  immediately on creation and keep them green. Why: clients read a green badge as proof the code is
  tested and trustworthy; stale red badges signal an unmaintained project. Example:
  ```text
  [![Build Status](https://github.com/ORG/REPO/actions/workflows/build.yml/badge.svg)](...)
  ```

### Additions from real codebases

- **Shared parent POM is the real convention carrier.** Every analysed repo inherits
  `com.jcabi:parent` (0.73.4 / 0.73.1 / 0.66.0) and keeps its own POM thin: quality tooling, Jacoco,
  release plumbing and compiler policy come from the parent (`knowledge/elegant-objects-java.md` §C, `knowledge/elegant-objects-java.md` §A,
  `knowledge/elegant-objects-java.md` §A). "Reading child POMs without the parent tells you little."
- **One non-interactive build command everywhere:** `mvn --errors --batch-mode clean install -Pqulice`
  (Cactoos `.github/workflows/mvn.yml:36`; Takes/Qulice/eo Rultor `merge`), documented verbatim in the
  READMEs. Cactoos profiles: `-Pqulice`, `-Pjacoco`, `-Psonar`; release adds `-Pcactoos -Psonatype`.
- **GitHub Actions = PR gate; Rultor = the only deployer.** rultor/jare make this explicit: no GHA
  workflow releases; merge/release happen via `.rultor.yml` chat-ops. Cactoos/Takes/eo/Qulice use the
  same split (Cactoos 13 workflows, Takes 14, eo **41**, Qulice 12).
- **Release is one tag-driven script.** `.rultor.yml` validates `^[0-9]+\.[0-9]+\.[0-9]+$`, imports
  GPG keys, runs `mvn versions:set -DnewVersion=${tag}`, commits `${tag}`, builds
  `mvn clean deploy -Psonatype`, and (rultor) stamps the git short hash into `META-INF/MANIFEST.MF`
  (`.rultor.yml:22-33`; eo adds `flatten:flatten`).
- **Secrets never touch history.** Assets are fetched from a separate `yegor256/home#assets` repo,
  marked `sensitive: settings.xml`; the release copies → commits → pushes → `git reset HEAD~1` + `rm`;
  `deploy.sh` wraps the same in `trap '...' EXIT` so cleanup happens on every exit path
  (`rultor/.rultor.yml:9-13,23-24,31-39`; `req/release.sh:37-43`; jare `deploy.sh:12`).
- **Deploy = git-push to Dokku/Heroku, then a live smoke test.** `git push -f heroku <branch>:master`
  followed by `curl -f --connect-timeout 15 --retry 5 --retry-delay 30 <live-url>`; site deploy is
  best-effort (`|| echo 'Failed to deploy site'`) (rultor/jare/s3auth/rehttp).
- **Integration infrastructure is provisioned per profile, not mocked.** `dynamodb` profile unpacks
  `DynamoDBLocal`, reserves a port via `build-helper:reserve-network-port`, and runs
  `jcabi-dynamodb-maven-plugin` start/create-tables/stop; `takes-test` boots the real app
  (`exec-maven-plugin`) for `*ITCase` failsafe runs (rultor `pom.xml:775-865`, rehttp
  `pom.xml:260-356`).

## 10. Project and repository management

*Not covered by this source.* Volume 1 contains no material on tickets, commit hygiene, branching,
release scripting, semantic versioning, GitHub releases, or repository conventions. It offers only a
general maintainability mindset (B8.2) and documentation advice ("document external interfaces, not
internals" — B4.14 / B8.4), which project-management agents can build on from other sources.


### Additions from articles

- **Acoa.39 Optimise for the project's objectives, not for pleasing the boss** -> add to §10: Every
  person (developer, PO, CTO) is a stakeholder; a professional "obeys and resists" — follows the
  process, and pushes back on any instruction that contradicts the project's objectives. Why:
  "making your boss happy" is a false objective that ruins projects; the project pays the checks.
  No snippet.

- **Acob.44 Measure programmers with quality-controlled, multi-factor metrics** -> add to §10: prefer Features Delivered, PRs Merged, Bugs Fixed, Bugs Reported, Releases Published, Uptime (MTBF/MTTF/failure rate), Cost of Pull Request, Documentation Pages Published, and Mentee Results over lines of code. Why: LoC and hours are bad metrics, but the absence of good metrics hurts more; every metric needs a verification mechanism (e.g. an architect rejecting duplicate/low-quality bug reports), because "trust without control leads to cheating". Closing/reviewing must be done by someone other than the author.
  ```text
  Metrics: features, PRs, bugs fixed/reported, releases, uptime,
  PR cost (time-to-merge), docs published, mentee results.
  Each must be validated (duplicates rejected, quality checked).
  ```

- **Atq.59 Every ticket is a complaint (Bug-Driven Development)** -> add to §10: Formulate bug reports, feature requests, and questions uniformly as complaints ("PNG downloading is broken, getting CSV instead"). Why: complaints force precise arguments and prevent noise; every ticket should describe a flaw in tangible source code and end in a patch. Example: use complaint-style titles, not "CSV" or "How can I download PNG?".

- **Atq.61 Minimize bug reports to the simplest scenario** -> add to §10: Reduce a bug report to the smallest reproduction that still shows the defect before filing. Why: reporter-side minimization (e.g. showing only the subtraction operator is broken) saves the maintainer's investigation time; an unminimized report is a valid rejection reason. Example:
  ```
  a := 7
  a := a - 3
  print a   // got 7, expected 4 => subtraction is broken
  ```

- **Atq.62 No "BTW" in a ticket discussion** -> add to §10: Keep a ticket to its original contract; any tangential bug, question, or feature request must become a new ticket. Why: scope creep distracts the team and blurs focus; they want to close the ticket, not chat. Example: reply "Good catch — please report that separately" instead of answering the aside.

- **Atq.63 Make bugs welcome and pay for discovering them** -> add to §10: Reward every reproducible, non-duplicate defect report that references existing functionality and can be fixed in reasonable time. Why: making hidden bugs visible before customers see them raises product quality; planning a bug count gives testers a goal. Example: pay 15 minutes for every accepted bug found.

- **Atq.64 Require either bug reports or pull requests from contributors** -> add to §10: A programmer contributes only by reporting bugs or submitting PRs; anything else is non-performance. Why: these are the only two ways a project advances, and they are measurable. Example: a contributor who neither files bugs nor opens PRs has no output.

- **Atq.65 Let programmers chase speed, let the project enforce quality** -> add to §10: Programmers should cut corners, make small changes, and not hesitate to break things; the pipeline is responsible for rejecting anything that lowers quality. Why: the deliberate conflict between fast delivery and an enforced quality wall yields a fast-growing, high-quality product. Example: a pre-flight build + read-only master makes it safe for developers to move quickly.

- **Adt.55 Tie every change to a GitHub issue and a same-named branch** -> add to §10: Work starts from an assigned issue; fork, `git checkout -b 123`, commit `#123: description`, push, PR, review, then bot-merge. Why: linear, traceable history, read-only master, no direct write access; discussion stays in the ticket. Example:
  ```bash
  git checkout -b 123
  git commit -am '#123: the description of the changes'
  git push origin 123
  ```

- **Adt.57 Let an automated merge bot merge and report in the PR** -> add to §10: Reviewer approves, then the merge bot merges into master, runs tests in a container, pushes, and comments. Why: the merge is consistent, validated, and logged in the ticket; the author is responsible until merged.

- **Ada.34 Break large repositories/modules into small ones** -> add to §10: Aggressively decompose; the largest acceptable code base is about 50,000 lines, and everything larger is a candidate for splitting. Why: small scope means easier maintenance, reuse, testing, and task assignment — exactly the argument for object decomposition applied to repos. Example (text):
  ```text
  monorepo (bad) -> many small repos (good):
  one language, one build, one issue tracker, one release each.
  ```
  *(also Ada.36)*

- **Ada.35 Decompose work top-down by user value** -> add to §10: Split a task into increments that each deliver value the customer can perceive, not into technical layers (tests first, product later). Why: the user pays for working features; a release that only adds a Makefile or a test suite provides zero value to them. Example (text):
  ```text
  Increment 1: skeleton + tests (user runs it, sees "5").
  Increment 2: real word counting.
  ```

- **Acs.34 Prefer distributed, async collaboration for cost and quality** -> add to §10: Run development as an extremely distributed team communicating through issues, not chat/desk talk, and pay by completed task. Why: a measured comparison found distributed work cost ~$0.13/line versus ~$3.98/line co-located (30x), because strict quality principles plus written communication replace costly informal overhead. Example (text):
  ```text
  20 co-located devs, 3 months: 88k changed lines for ~$350k  (~$3.98/line)
  15 distributed devs, 3 months: 54k changed lines for ~$7k   (~$0.13/line)
  ```

- **Apa.39 Mark every unknown in code with a `@todo` puzzle that names its origin ticket** -> add to §10: Use `// @todo #123 <description>` exactly where the code hits a stub, with enough detail for a stranger to implement it. Why: puzzles decompose an oversized task into parallelizable subtasks and make incomplete work a legitimate deliverable. Example:
  ```java
  // @todo #123 I assumed insert() is enough to save data. Maybe
  //  transactions are needed. Implement this interface and add an
  //  integration test with a database.
  ```

- **Apa.40 Place one puzzle per unresolved point, as close to the stub as possible** -> add to §10: Three failing test methods → three separate puzzles; put the puzzle next to the code it concerns. Why: proximity and granularity make each puzzle independently assignable and comprehensible. Example:
  ```text
  // don't write one vague todo; write N precise todos beside each ignored test.
  ```

- **Apa.41 Write puzzles as a complete specification for the next person** -> add to §10: The puzzle must say what to implement, how, which docs to use — enough that no follow-up to the author is needed. Why: a puzzle becomes someone else's task definition; hand-offs must need zero extra input. Example:
  ```text
  // @todo #123 Apply this transformation to the AST; see the Requs spec link;
  //  test with the fixture in src/test/resources/example.req
  ```

- **Apa.42 Run PDD mechanically with a GitHub bot** -> add to §10: A 0pdd-style bot scans master for `todo` markers, opens an issue per marker, closes issues when markers disappear, and publishes a badge/report. Why: automation scales PDD and keeps the puzzle backlog truthful without a manager. Example:
  ```text
  webhook: https://www.0pdd.com/hook/github (push event)
  every @todo → new GitHub issue; removed @todo → issue closed.
  ```

- **Apa.43 Adopt the No-Obligations Principle** -> add to §10: Reject any task you cannot finish; the manager may reclaim any task not delivered within ~10 days with zero pay. Why: it keeps responsibility on the performer and removes excuses and micromanagement. Example:
  ```text
  If not delivered in 10 days, PM takes the task away and pays nothing,
  regardless of hours already invested.
  ```

- **Apa.44 Start a task only when you are sure you can finish it** -> add to §10: Prefer fewer, completable tasks; ask all clarifying questions *before* starting. Why: only closed deliverables pay; starting an unclosable task wastes everyone's budget. Example:
  ```text
  Before work: ask the author every question; if unsure you can finish — decline.
  ```

- **Apa.45 Define Done as the author's acceptance and pay only for closed tasks** -> add to §10: A task is done iff its author accepts the deliverable; unfinished work is unpaid, even if it consumed days. Why: paying for deliverables (not time) aligns effort with results. Example:
  ```text
  Definition of Done = author accepts. Payment = per closed task, fixed time budget,
  independent of actual hours.
  ```

- **Apa.46 Fix and break at the same time** -> add to §10: While implementing your task, deliberately report new bugs and leave puzzles; total quality rises with every reported defect. Why: more found bugs → higher quality; every team member should both repair and expose issues. Example:
  ```text
  Deliverable = fixed code + new tests + at least one reported bug / puzzle.
  ```

- **Apa.47 Treat any inconsistency as a bug** -> add to §10: Unclear docs, missing tests, refactorable code, poor performance — all are bugs and all get tickets. Why: "bugs are welcome" and unlimited; the Fixing phase is mostly bug discovery, and that drives quality. Example:
  ```text
  "The design doc doesn't compare NoSQL options" → bug ticket → update documentation.
  ```

- **Apa.48 Report bugs on the way; one bug per 2–3 completed tasks is healthy** -> add to §10: Encourage (and pay for) bug reports alongside normal development. Why: bug reporting is the "breaking" half that raises final quality; top performers submit bugs continuously. Example:
  ```text
  Best developers: ~1 bug reported per 2-3 tasks closed.
  ```

- **Apa.50 Never grant write access; only pull requests** -> add to §10: No collaborator pushes to master, ever, no matter their tenure. Why: PR-only merges make every change reviewable, testable and traceable. Example:
  ```text
  remove everyone from "collaborators"; all changes via fork → PR → merge bot.
  ```

- **Apa.51 Communicate only through tickets; no email, chat or meetings** -> add to §10: Every question, decision and status lives in a tracking-system ticket addressed to the task author. Why: ticket-only communication is traceable, searchable, billable and available 24/7 across time zones. Example:
  ```text
  No Slack / Skype / email / calls. A question = new ticket; closed ticket = paid answer.
  ```
  *(also Atq.66, Adt.56, Apb.40)*

- **Apa.52 Keep the task author as the single point of contact** -> add to §10: No horizontal chatter about scope; the author is the only customer and clarifies requirements in the ticket. Why: one channel prevents responsibility leakage and duplicate/conflicting instructions. Example:
  ```text
  Receive task → ask author in ticket → deliver → author closes → PM pays.
  ```

- **Apa.53 Escalate complexity into new tickets, never into the current one** -> add to §10: If the task is blocked or unclear, submit dependent tickets (questions, docs, design bugs) and stop legally. Why: it reveals maintainability issues instead of hiding them and keeps the current task small and closeable. Example:
  ```text
  Main ticket: "blocked until dependencies #a, #b, #c are resolved."
  ```

- **Apa.54 Run projects in four phases: Thinking, Building, Fixing, Using** -> add to §10: Spend the most care on Thinking (spec), let Building be one architect's short prototype, run Fixing as mass bug work, and keep Using quiet. Why: a mistake in Thinking costs far more than later; Fixing is >70% of effort and where quality is won. Example:
  ```text
  Thinking: 2d-3w, spec. Building: 2-5d, one architect, working software.
  Fixing: weeks-months, many people, bug fixes via PRs. Using: feedback → bugs.
  ```

- **Apa.55 Write a short Product Vision in four sections** -> add to §10: Product statement, stakeholders & needs, actors & features, quality requirements — ~2 pages, ~1 minute read. Why: a short, explicit vision keeps everyone on the highest abstraction and prevents feature bloat. Example:
  ```text
  Statement / Stakeholders+Needs / Actors+Features / Quality Requirements
  (each section ≤ 20 lines; 3-6 features per actor; ≤ 6 quality requirements)
  ```
  *(also Apa.97)*

- **Apa.56 List both positive and negative stakeholders** -> add to §10: Name everyone affected, including those who lose (e.g. staff made redundant), and state 1–2 needs each. Why: success = satisfy positive stakeholders and neutralize negative ones. Example:
  ```text
  Sponsor: raise investments. Users: share photos. Government: protect society.
  Competitors: wipe us off the market.
  ```

- **Apa.57 Specify requirements incrementally as tiny SRS fixes** -> add to §10: Treat each SRS defect as a bug ("UC1 doesn't explain how a raise happens"); fix it in place, one small change per ticket. Why: small increments keep requirements complete and unambiguous without a frozen doc. Example:
  ```text
  Department has employee-s. Employee has name and salary.
  UC1 where Employee gets raise: "TBD."
  ```

- **Apa.58 Use a controlled natural language for the SRS** -> add to §10: Write requirements in a parseable CNL (Requs) composed from `.req` files under `src/main/requs`, compiled by CI. Why: machine-checkable specs stay consistent and update incrementally. Example:
  ```text
  mvn clean requs:compile   → BUILD SUCCESS required before PR
  ```

- **Apa.59 Ban questions, discussions, suggestions and opinions from specs** -> add to §10: Put facts only; if unknown, write `TBD`; move debate to a separate channel and delete its history. Why: a spec is a contract, not a brainstorm; opinions and hedging only confuse implementers. Example:
  ```text
  WRONG: "I believe multiple API versions must be supported. What options do we have?"
  RIGHT: "Multiple versions of the API must be supported. How exactly doesn't matter."
  ```

- **Apa.60 Never put implementation instructions in a spec** -> add to §10: Require behavior ("user logs in via Facebook"), not mechanism ("click a button, store email in DB"). Why: mechanism is micromanagement; the team chooses DB, placement and storage. Example:
  ```text
  WRONG: "User authenticates via Facebook login button and we store email in the database."
  RIGHT: "User authenticates via Facebook." (+ measurable constraints if business-critical)
  ```

- **Apa.61 Split functional requirements from supplementary detail** -> add to §10: Keep functional requirements to one short verb phrase ("user downloads"); put report layout etc. in an appendix. Why: long functional statements are a smell and become hard to trace and modify. Example:
  ```text
  Functional: The user downloads a PDF report.
  Appendix: report contains ID, date, description, account, amount, summary, link.
  ```

- **Apa.62 Appoint exactly one accountable architect** -> add to §10: The architect takes personal blame for technical quality and owns all final technical decisions. Why: shared/absent ownership destroys quality; responsibility must come with power. Example:
  ```text
  ARC: approves all PRs before merge; reports to PM; decisions final; replace if quality fails.
  ```

- **Apa.63 Demand a regular architect report: scope, issues, risks** -> add to §10: Every few days send a PBS (4–8 items with %), 4–8 issues, and 4–8 risks each scored [probability x impact] on 0–9. Why: it keeps the sponsor able to predict and decide, and makes omissions a firing reason. Example:
  ```text
  Scope: 1 MySQL persistence [done]  2 OAuth [done]  3 XML parsing [75%]
  Issues: 1 MySQL too slow  2 Java 1.6 blocks lib X
  Risks:  1 Lucene may not handle billions [6x9]  2 Platforms may ban us [8x9]
  ```

- **Apa.64 Run the architect on two instruments: bugs and reviews** -> add to §10: The architect encodes vision as bug tickets (proactive) and as PR approvals/rejections (reactive). Why: in a meeting-free, ticket-only team, tickets are the only legitimate way to direct technical work. Example:
  ```text
  Bug: "Design docs lack a NoSQL comparison; fix."  Review: reject PR violating architecture.
  ```

- **Apa.65 Never make the architect convince a stakeholder** -> add to §10: The architect decides and owns the outcome; a stakeholder who disagrees changes the requirements or files an architecture bug. Why: convincing splits responsibility ("I was forced") and destroys accountability. Example:
  ```text
  Disagreement with Maven? PO edits requirements to "must be Gradle"
  or files a bug about the unchallenged decision — not a debate with the architect.
  ```

- **Apa.66 Organize systematic independent technical reviews** -> add to §10: Hire an outside reviewer monthly, for 2–8 hours, starting from the repo's first file; never let reviewers talk to the team. Why: independent eyes expose architecture/design flaws and keep the sponsor's picture of quality honest. Example:
  ```text
  Monthly: reviewer checks out the code cold, reports ~20 most critical issues
  through the tracker only, then leaves.
  ```
  *(also Adt.58, Apb.59)*

- **Apa.67 Pay reviewers per bug found, not per hour** -> add to §10: Budget ~15 min per reported bug (~4/hour); pay only for accepted reports. Why: it focuses the review on finding issues, its only goal, and avoids billing for pleasant conversation. Example:
  ```text
  Rate $150/h, target 20 bugs → estimated 5h → reviewer is paid $750 for 20 reported issues.
  ```

- **Apa.68 Rotate independent reviewers; ask for criticism, not praise** -> add to §10: Never reuse the same reviewer on one codebase; ask "what should we fix first?", review everything (not only source), and track resolution. Why: familiarity breeds team-membership and hiding issues; flattery tells the sponsor nothing. Example:
  ```text
  Questions asked: "What problems should we fix first?" NOT "Do you like our code?"
  Review scope: code + schema + CI + tracker + plans + logs + bug reports.
  ```

- **Apa.69 Bill work incrementally against a stream of micro-tasks** -> add to §10: Weekly bill lists every delivered increment with its cost; each increment is 30–60 min of work; provide a re-estimated plan. Why: it removes fixed-price/T&M risk and pays only for verified, quality-checked deliverables. Example:
  ```text
  Weekly invoice = [increment → time → cost] × N  +  updated plan/budget forecast
  ```

- **Apa.70 Estimate cost as price-per-unit, never as a fixed total** -> add to §10: Refuse "how much to build X?"; answer "how much software per $100 and at what quality", backed by regular checkpoints. Why: software never finishes; only rate × quality × control gives a real guarantee. Example:
  ```text
  WRONG: "Hey driver, how much to get there?"
  RIGHT:  "How much per mile, and do you have a map?" (HoC, bugs, coverage, releases)
  ```

- **Apa.71 Track Hits-of-Code, not lines-of-code** -> add to §10: Count every line modified/created/deleted in Git; use the monotonically growing number to gauge effort. Why: HoC always rises with effort, is objective about custom vs vendored code, and captures line complexity. Example:
  ```text
  $ gem install hoc && hoc
  54687
  ```
  *(also Acs.35)*

- **Apa.72 Confirm the estimate with checkpoints, milestones and release notes** -> add to §10: Insist on small increments, independent reviews, milestones, versioned releases and published notes. Why: these are the only mechanisms that let a sponsor steer and correct the work in progress. Example:
  ```text
  Checkpoint = reviewable increment + review + versioned release + notes.
  ```

- **Apa.73 Write down every key decision with its alternatives** -> add to §10: In `README.md` list 4–12 decisions, each with at least one considered-and-rejected alternative; note the responsible person. Why: traceable decisions expose who chose what and why, improving future judgment. Example:
  ```text
  Lucene is the search engine (alternatives: Solr, Sphinx, Gigablast).
  Java 8 runtime (alternatives: Ruby, Python, Go, Scala).
  ```

- **Apa.74 Record assumptions, risks and concerns explicitly** -> add to §10: Document gaps you filled, list 4–12 risks scored [prob x impact], and answer every requirement concern honestly. Why: documenting uncertainty makes it manageable and reviewable instead of hidden. Example:
  ```text
  Assumptions: social APIs won't block us; Lucene suffices without a DB.
  Risks: Lucene may not scale [6x9]; platforms may ban us [8x9].
  Concerns: the system is as fast as Lucene; horizontal scalability unknown.
  ```

- **Apa.75 Set explicit, written ground rules before any reward or punishment scheme** -> add to §10: Publish how results are measured, who assigns tasks, deadlines, quality expectations, and how mistakes affect grades. Why: money/awards demotivate without rules; the rulebook must be superior to any boss. Example:
  ```text
  Staffing management plan answers: how measured? who assigns? how to resolve task conflicts?
  deadlines per task? quality expectations? effect of mistakes?
  ```
  *(also Apb.63)*

- **Apa.76 Make all performance decisions derive from rules, not personal judgment** -> add to §10: Fire, raise and reassign strictly per published rules and open reconciliation; never behind closed doors. Why: rule-derived decisions are accepted even by the affected person and prevent dictatorship. Example:
  ```text
  Not: "I don't like him."  But: "He doesn't meet performance rule R4."
  ```

- **Apa.77 Keep the firing decision impersonal and rule-derived** -> add to §10: Firing should be the logical consequence of agreed rules, visible to the team, not a manager's emotion. Why: if firing feels painful and hidden, the rules are unclear and management is flawed. Example:
  ```text
  Firing is derived from a rule violation; the team understands and supports it.
  ```

- **Apa.78 Make responsibility strictly personal, never "together"** -> add to §10: Assign each task to one person who owns success/failure; the manager owns only how parts combine. Why: "together" is the most demotivating word; it signals the manager is shifting responsibility away. Example:
  ```text
  Each task = one owner. Manager owns the assembly of outcomes, not the outcome of each part.
  ```

- **Apa.79 Define motivation as objectives with awards, penalties and rules** -> add to §10: Translate company goals into personal objectives with explicit upside/downside, and delegate accordingly. Why: a fully specified objective removes the need for daily status updates and micromanagement. Example:
  ```text
  "Deliver the feature before the weekend → company profit and you personally get $500;
   fail → you move to a less interesting project."
  ```

- **Apa.80 Replace status meetings with a communication model** -> add to §10: Never run daily stand-ups or status meetings; team members report immediately when necessary through the tracker. Why: stand-ups are a micro-manager's tool, demotivate A-players by enforced equality, and manufacture guilt. Example:
  ```text
  No stand-ups. Good manager: defines awards/penalties/rules; team informs when needed.
  ```

- **Apa.81 Reach decisions by written documents circulated through PRs, not meetings** -> add to §10: Ask a colleague to draft a document, iterate via pull requests and comments; the Git history preserves reasoning forever. Why: meetings burn everyone's time, demotivate and leave no searchable artifact; documents do. Example:
  ```text
  schema.md → Jeff opens PR with draft + Assumptions/Risks/Concerns
             → Monica reviews with a new PR → merged after review.
  ```

- **Apa.82 Move up the communication-maturity ladder toward repo-integrated channels** -> add to §10: Avoid coffee breaks, calls, meetings, email, mailing lists and Slack; prefer ticket trackers and finally GitHub where talk and code co-locate. Why: the farther a channel is from project artifacts, the more information is lost. Example:
  ```text
  coffee < calls < meetings < email < lists < Slack < Trello < GitHub
  (GitHub best: discussion lives beside the code)
  ```

- **Apa.83 Keep turnover high enough to prevent heroes and strong code ownership** -> add to §10: Rotate programmers away from a codebase before they become single points of knowledge. Why: long-tenured experts create hero-driven development, hide issues, and make the code unmaintainable without them. Example:
  ```text
  Rotate before ~1 year per codebase; prioritize collective code ownership.
  ```

- **Apa.84 Manage the project with autocracy, the organization with holacracy** -> add to §10: Use a hierarchy with awards/punishments/rules inside each temporary project; use flat democracy only for the enduring organization. Why: a project must end ("discipline, subordination, rules"), while a company must survive ("tolerance, respect, equality"). Example:
  ```text
  Project: one PM as dictator. Company: flat team. Matrix org = both.
  ```

- **Apa.85 Let the project manager predict, not build** -> add to §10: The PM organizes resources so the future is predictable and reports when to kill the project; the team builds. Why: sponsors need an accurate prediction to decide keep-vs-kill; a PM who "becomes the future" micromanages. Example:
  ```text
  PM: "total cost expected $1.09; probability of success 87.4%."  Team: does the work.
  ```

- **Apa.86 Keep order and status in a project management information system** -> add to §10: All communication flows through the PMIS; plans, work orders, risks and concerns are explicit and self-service. Why: a perfect PM is nearly invisible; work orders are created/assigned/verified by the team itself. Example:
  ```text
  Plans available, work orders defined, risks documented, stakeholders informed in time.
  ```

- **Apa.87 Distinguish an issue from a risk** -> add to §10: An issue already happened; a risk may happen; report both, using "may" for risks and probability×impact scoring. Why: conflating them hides future trouble or wastes attention on the past. Example:
  ```text
  Issue: "MySQL is too slow [now]."   Risk: "MySQL may not scale [4x8, future]."
  ```

- **Apa.88 Cut corners professionally: create dependencies before heroics** -> add to §10: When blocked, first file bugs about unclear design/missing tests, then demand docs, then reproduce with a test, then disable, then say no. Why: the disciplined priority order protects nerves, budget and code quality and reveals problems instead of hiding them. Example:
  ```text
  1 create dependencies (blame the project)  2 demand better documentation
  3 reproduce with a test (and @Ignore)       4 disable the feature  5 say "no"
  ```

- **Apa.89 Blame the project for missing skills; demand documentation as the fix** -> add to §10: Instead of asking for training, file a documentation bug the project will pay to fix; learn from the improved artifact. Why: it aligns personal learning with project value and avoids spending project money on a school. Example:
  ```text
  "Docs are not detailed enough for a Java dev to build this Python module; please fix."
  ```
  *(also Apb.62)*

- **Apa.90 Keep roles explicit: PM, PO, ARC, DEV, REQ, QA, TST** -> add to §10: Name who owns control, requirements, technical solution, closing bugs, validation and process compliance. Why: in a distributed, ticket-only project, clear roles replace proximity and supervision. Example:
  ```text
  ARC approves every PR; QA approves every closed task before PM closes it; all roles may file bugs.
  ```

- **Apa.91 Never confuse project management with leadership** -> add to §10: Value the PM who predicts and organizes over the charismatic leader who makes people obey. Why: charisma replaces rules with personality and destroys the management system. Example:
  ```text
  A perfect PM needs no leadership skills; a lousy PM needs a lot of them.
  ```

- **Apa.92 Delegate through goals, not algorithms** -> add to §10: State the expected result and the rules for success/failure; never dictate the steps. Why: telling *how* is micromanagement and treats people as dumb executors; telling *what* respects professionalism. Example:
  ```text
  GOOD: "The Nginx server must be up by 6 p.m. I'm counting on you."
  BAD:  "Install Nginx now and nothing else until it's done."
  ```

- **Apa.93 Put money on the table with explicit rationale** -> add to §10: Make salaries, bonuses and promotion criteria public, each with a stated reason and a path to increase. Why: hidden compensation spawns rumor and politics; explicitness makes competition fair and motivating. Example:
  ```text
  Public: everyone's salary, bonus logic, and "do X to earn a $5,000 raise";
  the rationale is printed on the wall behind the manager's chair.
  ```

- **Apa.94 Define who leaves first when the project fails, in writing, in advance** -> add to §10: A team-growth plan must state risks, mitigations, and the order of exits before trouble comes. Why: a predictable manager and plan make the team feel secure and respects each member's market value. Example:
  ```text
  Plan: includes risks + mitigation + who will be fired first if the project goes down.
  ```

- **Apa.95 Talk about money only after results, and treat unpaid effort as worthless** -> add to §10: Payment follows verified deliverables; no credit for time invested without a closed task. Why: time-based pay is "modern slavery"; selling results is the free/professional model. Example:
  ```text
  "Your efforts are not appreciated — only the deliverables matter."
  ```

- **Apa.96 Make the project end, and convert big new features into new projects** -> add to §10: The Using phase accepts only bug fixes; big feature requests spin off a fresh Thinking→Building→Fixing cycle. Why: scope creep in maintenance is how projects become endless and unmaintainable. Example:
  ```text
  New big feature → new repository / new project starting at Thinking.
  ```

- **Apa.98 Cap features per actor and quality requirements per product** -> add to §10: 3–6 features per actor; ≤ 6 quality requirements; if more, group or split. Why: bounded lists force understanding and expose feature bloat. Example:
  ```text
  User: create account, upload photos, share photos, message, like, purchase likes (6).
  ```

- **Apb.30 Every idea starts with a ticket** -> add to §10: Require a GitHub issue before any code,
  review or decision, and keep all discussion in the ticket. Why: it replaces informal chat with a
  traceable record and is one of the architect's enforceable rules ("no informal discussions outside of
  tickets"). Example:
  ```text
  Rule: Every idea starts with a ticket; no changes without code review.
  ```

- **Apb.31 Break scope into 30-minute micro-tasks** -> add to §10: Decompose the project into tasks of
  roughly 30 minutes so each can be delegated and closed under a 0/100 rule (in-progress or done, never
  in between). Why: micro-scope makes a project manageable and lets you trust performers without
  micromanaging; bigger tasks lose manageability and force micromanagement. Example:
  ```text
  "Redesign The Apartment" -> 2,500+ microtasks of 30 minutes each
  ```
  *(also Apa.49)*

- **Apb.32 Fixed micro-budget per task, PDD for overflow** -> add to §10: Give every task a small fixed
  budget (e.g. 30 min); allow the performer to complete only part of it and return `todo` puzzles that
  are auto-converted into new tasks. Why: it moves scope decomposition to the people best at it
  (programmers) and makes pricing predictable. Example:
  ```text
  // @todo #42:30min Extract Line into its own class
  // 0pdd converts each puzzle into a new ticket automatically
  ```

- **Apb.33 Define "done" and the consequences explicitly** -> add to §10: Put the definition of done,
  the reward and the punishment in writing for each task before work starts. Why: clear, explicit
  rules (payment policy, consequences of breaking master, missing a deadline) remove fear and boost
  motivation far more than vague promises. Example:
  ```text
  Task: fix bug #17 — done = PR merged + test green; reward = $25; late >3d = unpaid
  ```

- **Apb.34 Proof-of-concept and pipeline before hiring** -> add to §10: Don't invite programmers until
  the prototype proves the key technical objective, the pipeline is configured and the product is
  deployed to production. Why: adding people to an unproven, undeliverable base multiplies waste;
  confirm the riskiest assumption first. Example:
  ```text
  1) prototype 2) green pipeline 3) prod deploy 4) THEN recruit the team
  ```

- **Apb.35 Read the README as the showcase** -> add to §10: Ship a README whose first paragraph states
  what the product IS, followed by "how to try it" and "how to contribute," and nothing about license,
  changelog or contributors. Why: the README is where every visitor decides in ~60 seconds whether to
  stay; it must shine and stay short/high-level. Example:
  ```text
  logo -> badges (<=5/line, equal heights) -> one-paragraph "what is it" ->
  "how to use it" -> use cases -> "how to contribute"
  ```

- **Apb.36 State the problem in the README's first paragraph** -> add to §10: The README must answer
  "What is wrong with the world now, and how does this product fix it?" in its first paragraph. Why: a
  team without a clearly stated problem loses morale and direction, and the problem statement is the
  leader's primary responsibility. Example:
  ```text
  "X is broken because Y. This tool does Z to fix it."
  ```

- **Apb.37 Follow a strict README formatting canon** -> add to §10: Keep README lines ≤80 chars (except
  badges), use no indentation, second-level headers only for sections (avoid `###`), one blank line
  between blocks, and redirect detail to auto-generated docs. Why: the README is a piece of source code
  that must stay elegant, short and cheap to maintain. Example:
  ```text
  ## Use cases
  (no indentation, <=80 cols, link to Javadoc instead of prose)
  ```

- **Apb.38 One change = one small pull request** -> add to §10: Never bundle unrelated changes; keep
  PRs small (<50 hits-of-code for newcomers) and focused, since review quality drops with size. Why:
  Google research and the major open-source guides all show large PRs review worse and merge slower.
  Example:
  ```text
  A PR titled "[bug] null guard in PriceTest" contains only that fix.
  ```
  *(also Atq.60)*

- **Apb.39 Address comments with the person's nickname** -> add to §10: Start every issue/PR comment
  with `@nickname` of the person you are talking to, and ping on new issues, new PRs and after pushes.
  Why: GitHub only notifies the addressed user; without it the message is silently lost among hundreds
  of emails. Example:
  ```text
  "@architect please take a look at this PR when you have a minute."
  ```

- **Apb.41 Let the reporter close the ticket** -> add to §10: After fixing a reported bug, ask the
  reporter to verify the fix and let them close the ticket; only they should close it. Why: closing
  unilaterally feels dismissive, suppresses the tester-programmer quality conflict, and spawns
  duplicate tickets. Example:
  ```text
  Fix -> comment "@reporter please confirm and close" -> reporter closes.
  ```

- **Apb.42 Exception rules for closing tickets** -> add to §10: Close immediately only if the ticket is
  an obvious duplicate, is a question answered in place, or carries a `won't fix` badge — otherwise let
  the reporter close. Why: this preserves the fairness of the process while avoiding pointless waits.
  Example:
  ```text
  duplicate -> close now; question -> answer + close; won't fix -> badge + close
  ```

- **Apb.43 Report tangible results, not effort** -> add to §10: When a status report is required,
  list only visible artifacts ("ticket closed," "document created"), never hours or busyness. Why: it
  keeps reporting honest and measurable, and prevents the guilt-driven ambiguity of "what did you do
  today." Example:
  ```text
  - Added 100 files to the Dataset [100%]
  - Deployed XYZ to staging [50%]
  ```

- **Apb.44 Use SIMBA: Plan of artifacts with owner + reviewer** -> add to §10: Maintain a Plan whose
  every line is a deliverable artifact with an owner, a reviewer, a date and a subjective % complete;
  cap each person at ~3 owned and ~4 reviewed items. Why: tracking *artifacts and delivery status*
  (not tasks) makes progress visible and accountability explicit. Example:
  ```text
  - Requirements v1 [Jeff+Bill, 25-Aug, 80%]
  - Dataset with 500+ files [Anna+Jeff, 3-Sep, 100%]
  ```

- **Apb.45 Weekly report format: WEEKnn + <=7 wins/tasks + risks** -> add to §10: Send a Monday email
  whose subject starts with `WEEKnn`, listing at most seven past-tense achievements (with links) and
  seven infinitive next steps, plus Cause-Risk-Effect risks. Why: a short, searchable, verifiable
  report is the minimum communication that keeps sponsors and the team aligned; <=7 forces prioritization.
  Example:
  ```text
  Subject: WEEK13 Dataset, Requirements, XYZ
  Last week achievements: - Added 100 new files to the Dataset [100%]
  Next week plans:        - To publish ABC package draft
  Risks:                  - Weak server may cause a missed dataset milestone.
  ```

- **Apb.46 Hold weekly status calls (minutes) and demos** -> add to §10: Run a 30-min weekly call to
  inspect the Plan (not to report), email the Meeting Minutes, and hold a weekly one-hour demo of a
  ready artifact; record and archive calls. Why: status calls decide, reports inform, and demos give
  stakeholders the "look at it" trust that numbers cannot. Example:
  ```text
  Weekly call -> Meeting Minutes by email
  Weekly demo -> recorded, posted to the team's private list
  ```

- **Apb.47 Use double-blind review for proposals** -> add to §10: Route budget/project proposals to a
  board of 10+ via a secretary who randomly picks three anonymous reviewers who submit written
  evaluations; accept on 2+ endorsements without revealing reviewer names. Why: anonymity plus written
  authorship removes friendship, jealousy, fear and incompetence bias from decisions and raises
  proposal quality. Example:
  ```text
  Secretary -> random 3 reviewers -> written feedback -> 2/3 accept -> anonymous verdict
  ```

- **Apb.48 Decompose and delegate consequences, not just scope** -> add to §10: When delegating an
  objective, hand over the proportional reward/punishment as well as the work; a flat salary plus a
  token bonus does not do it. Why: people worry only as much as the consequence is theirs; a mismatch
  leaves the leader carrying all the risk ("you are the low-hanging fruit"). Example:
  ```text
  Bad:  $5/hour + promises
  Good: "$40 when the list is complete and I accept it"
  ```

- **Apb.49 Make them chase you (inversive management)** -> add to §10: Put a price tag on each needed
  result and become the buyer; let performers chase you to collect payment. Why: turning yourself into
  a buyer makes them own the deadline, requirements and quality, instead of you chasing their effort.
  Example:
  ```text
  Instead of "$5/hour", offer "$40 when the list is done".
  ```

- **Apb.50 Reward results, taboo time-based pay** -> add to §10: Bind compensation to measurable
  merged results (merged PRs, closed tasks) and forbid discussion of time-based compensation. Why:
  incentives shape behavior — "if you reward excuses you buy excuses; if you reward results you get
  results." Example:
  ```text
  Pay = f(merged pull requests); no hourly rate is offered or discussed.
  ```

- **Apb.51 Let the team plan from a reward menu** -> add to §10: Publish deliverables with reward tags
  and ask each programmer how much they want to earn; derive the plan from their declared intentions.
  Why: greed-based planning replaces the dishonest top-down commitment dance and yields a plan the team
  actually owns. Example:
  ```text
  "Here are 40 tasks with rewards. How much do you want to earn?"
  -> programmer: "I'll do these 12 for $5,000" -> that becomes the plan.
  ```

- **Apb.52 Measure each person's own input, hide the top line** -> add to §10: Evaluate people only on
  work they control; don't cascade company-level OKRs or parade the bottom line as motivation. Why:
  expectancy, instrumentality and equity theories plus Deming's red-bead experiment show that judging
  people on unmeasurable, uncontrollable outcomes kills motivation and invites sabotage. Example:
  ```text
  Visible: the worker's own artifact/task metric.
  Hidden:  revenue, fundraising, top-line goals.
  ```

- **Apb.53 Stop manager appraisal, keep the management system** -> add to §10: Replace subjective human
  performance appraisal with a system of objective metrics and automate the routine management; don't
  dissolve management into a manager-free "self-managing" chaos. Why: the root of most management
  complaints is biased appraisal, so take appraisal away from managers rather than removing management.
  Example:
  ```text
  Keep: budgeting, delegation, responsibility.
  Automate: who did what and how well (metrics/tools).
  ```

- **Apb.54 Make the measurement system public and peer-reviewed** -> add to §10: Publish a fixed list of
  rewardable achievements with point values, weight them by seniority, collect reports by "push," and
  let peers dispute entries openly. Why: transparent, weighted Calibrated Achievement Points reward the
  "extra mile" and reduce conflicts/cheating because everyone can see and challenge the ranking. Example:
  ```text
  Major Product Release: 30 (4/yr); Conference Article Accepted: 70; GitHub Star: 1
  points = value / personal weight (e.g. 70 / 10 = 7 for a junior)
  ```

- **Apb.55 Decompose trust continuously, not 0/100** -> add to §10: Grade people incrementally with
  micro-bonuses and micro-penalties after each iteration; never treat them as "fully trusted until
  fired." Why: binary trust is as dangerous as binary tasks — it hides drift and makes the eventual
  break-up arbitrary and humiliating. Example:
  ```text
  After each task/PR: small reward or small penalty; update reputation.
  ```

- **Apb.56 Reputation penalizes refusal, not just money** -> add to §10: Deduct reputation points (not
  cash) for refusing tasks so the system learns who is reliable; higher reputation earns first pick of
  work. Why: refusal usually means the person cannot solve a problem within scope, and the system needs
  that signal to make assignment decisions. Example:
  ```text
  Refuse a task -> small reputation hit (no wallet loss) -> fewer future assignments.
  ```

- **Apb.57 Five levers to speed a freelancer team** -> add to §10: To accelerate, use (1) more
  developers, (2) higher rates, (3) boost factor on important tasks, (4) shorter per-task window, and
  (5) eject slow producers. Why: with no obligations, only motivation and consequences move the needle;
  a 3-tasks-per-week freelancer is normal, so scale the team rather than begging individuals. Example:
  ```text
  boost important tasks 2x-3x; shorten the 10-day window; discharge the slow.
  ```

- **Apb.58 Hire QA to police the rules** -> add to §10: When a project starts, add a Quality Assurance
  role whose job is to prevent abuse of the rules of work. Why: without QA the team is at high risk of
  losing discipline and money; the role exists precisely to enforce the process. Example:
  ```text
  QA reviews that each microtask had its reward/punishment applied correctly.
  ```

- **Apb.60 Build a coalition to change a team** -> add to §10: To change entrenched practices, mentor
  the youngest/most ambitious, hold optional weekly lectures, push small refactorings and have the
  group review each other's PRs — assemble a minority that shifts the majority. Why: a lone reformer is
  sabotaged, but ~7 committed supporters among 30 changed the direction without firing anyone. Example:
  ```text
  mentor a few -> weekly lectures -> small refactor PRs -> mutual code review -> coalition
  ```

- **Apb.61 Propose a plan; never ask "what next?"** -> add to §10: After getting the goal, come back
  with a concrete plan for the manager to approve or reject, rather than asking "What should I do
  next?". Why: the question signals passivity and dependence; proactive planning is what marks a strong
  contributor. Example:
  ```text
  Bad:  "What should I do next?"
  Good: "Goal received. Here is my plan: A, B, C — approve?"
  ```

- **Apb.64 Praise/blame only for what a person controls** -> add to §10: Attribute reward and
  punishment only to a person's own inputs; never pin a team-level or uncontrollable outcome on them.
  Why: the control principle plus expectancy/equity theory show that judging uncontrollable results
  destroys motivation and provokes retaliation. Example:
  ```text
  Do:   reward "closed ticket #42 with green CI"
  Don't: blame one engineer for a missed quarterly revenue target.
  ```

### Additions from real codebases

- **SPDX header on every source and config file.** `SPDX-FileCopyrightText: Copyright ... Yegor
  Bugayenko` + `SPDX-License-Identifier: MIT` on `.java`, YAML, TOML and shell, enforced by
  `license-maven-plugin` at `verify` (`license-maven-plugin:check-file-header`) and the `reuse`
  GitHub Action (`fsfe/reuse-action`); `REUSE.toml` + `LICENSES/MIT.txt` map non-source globs (with
  `precedence = "override"` in rultor `REUSE.toml:49-51`).
- **`.gitattributes` is a deliberate cross-platform policy.** `* text=auto eol=lf`, `*.java ident`,
  `*.xml ident`, `*.png binary`; Qulice's own repo even pins `NewLines.java -text` and `newlines.txt
  -text` so its newline fixtures are never normalised.
- **Renovate, never Dependabot.** Every repo has `renovate.json` extending `config:base`; eo adds
  `ignoreDeps` + `allowedVersions` guard-rails so plugins that must not float stay pinned; rultor pins
  `ignoreDeps: ["xml-apis:xml-apis"]`. No `.github/dependabot.yml` exists in any repo.
- **PDD (Puzzle-Driven Development).** `.pdd` rules (`--rule min-words:20 --rule min-estimate:15
  --rule max-estimate:90`, or repo-specific), `.0pdd.yml` (tags `pdd`/`bug`, maintainer email), a
  `pdd.yml` workflow, and `install: pdd -f /dev/null` in Rultor. Puzzles are `@todo #NNNN:30min …`
  Javadoc tags (14 in Cactoos source); 0pdd turns them into GitHub issues.
- **README as the external contract.** Cactoos 489 lines, Takes 1,319 lines — badges (EO principles,
  Rultor, IntelliJ, mvn, codecov, PDD), install snippets (Maven + Gradle + required JDK), a usage
  cookbook, an explicit **Architecture** section stating the load-bearing design rules, contribution
  instructions with the exact Docker/`mvn` command, and a Contributors list.
- **Docs drift is fixed by a bot, not discipline.** `up.yml` fetches the latest git tag, `sed`-rewrites
  the version in `README.md`, and opens a signed PR via `peter-evans/create-pull-request` (cactoos,
  takes, eo, requs, rehttp, xembly).
- **Extra repo artifacts.** `CITATION.cff` (takes, requs, eo), `rultor_schema.json` + config docs
  (rultor), `src/www.takes.org` / `src/site` published to `gh-pages` (takes, xembly), LaTeX/UML class
  docs via `jcabi-latex-maven-plugin` (s3auth). No `CONTRIBUTING.md` or issue/PR templates in most
  repos — the contribution contract lives in the README.
- **`.gitignore` tracks AI-tool dirs.** `.claude/`, `.aider*`, `.aidy` are ignored in cactoos/eo —
  current-generation repo hygiene.

## 11. Tools and libraries

- **B11.1 Use Lombok `@EqualsAndHashCode` to implement state-based equality** — On immutable classes,
  generate `equals()`/`hashCode()` from the encapsulated state. Why: in EO, state is identity, so
  value equality must be implemented explicitly; Lombok removes the boilerplate.
  Example (Java):
  ```java
  @lombok.EqualsAndHashCode
  final class WebPage {
      private final URI uri;
      WebPage(URI path) { this.uri = path; }
  }
  ```

- **B11.2 Use JUnit + Hamcrest `assertThat` for single-statement tests** — Write tests as one
  `assertThat(actual, equalTo(expected))`. Why: the matcher form reads declaratively and supports the
  one-statement rule.
  Example (Java):
  ```java
  assertThat(new Cash("$5").plus(new Cash("$3")), equalTo(new Cash("$8")));
  ```

- **B11.3 Avoid Mockito and other mocking frameworks** — Prefer hand-written `Fake` classes shipped
  with interfaces. Why: mocking couples tests to implementation and produces false failures; fakes
  keep tests short and refactor-safe.
  Example (Java):
  ```java
  // discouraged
  Exchange exchange = Mockito.mock(Exchange.class);
  // preferred
  Exchange exchange = new Exchange.Fake(1.2345f);
  ```

- **B11.4 Use an AOP library (e.g. jcabi-aspects) for cross-cutting concerns** — Implement retries,
  logging and similar mechanisms as aspects (`@RetryOnFailure`). Why: AOP keeps supplementary
  techniques out of main classes and avoids duplication and premature exception recovery.
  Example (Java):
  ```java
  @com.jcabi.aspects.RetryOnFailure(attempts = 3)
  public String content() throws IOException { return http(); }
  ```

- **B11.5 Wrap static utility libraries in objects** — When you must use Apache Commons/Guava-style
  statics, hide the call inside a class (e.g. `FileLines` over `FileUtils.readLines`). Why: your code
  then depends on objects, and the static call is confined to one place that can later be removed.
  Example (Java):
  ```java
  class FileLines implements Iterable<String> {
      private final File file;
      FileLines(File file) { this.file = file; }
      @Override public Iterator<String> iterator() { return Arrays.asList(FileUtils.readLines(this.file)).iterator(); }
  }
  ```

- **B11.6 Avoid `java.util.Optional` as a null replacement** — See B6.14. Why: it is an envelope, not
  the represented object, and contradicts object thinking.
  Example (Java):
  ```java
  // discouraged: java.util.Optional<User> user(String name);
  // preferred: Collection<User> users(String name);
  ```

- **B11.7 Prefer `AutoCloseable` / try-with-resources to emulate RAII in Java** — Where C++ uses
  destructors, Java should use try-with-resources on `AutoCloseable`/`Closeable`. Why: RAII captures a
  resource for the object's lifetime and releases it deterministically; use it for files, streams and
  DB connections.
  Example (Java):
  ```java
  try (Text t = new Text("/tmp/test.txt")) {
      t.content();
  }   // close() is called here
  ```

- **B11.8 Ship `Fake` and `Smart` nested classes with interfaces as library API** — Interfaces in a
  library should come with nested `Fake` and `Smart` classes. Why: it makes the interface usable and
  testable out of the box, and attacks the amount of mocking in the ecosystem.
  Example (Java):
  ```java
  interface Exchange {
      float rate(String origin, String target);
      final class Fake implements Exchange { @Override public float rate(String o, String t) { return 1.2345f; } }
      final class Smart { /* convenience methods */ }
  }
  ```


### Additions from articles

- **Acoa.40 Use the jcabi family and OO primitives over Apache Commons/Guava** -> add to §11:
  `jcabi-http`, `jcabi-xml` (`XSLDocument`), `jcabi-github` (`RtGitHub`/`MkGitHub`), `jcabi-s3`
  (`Ocket`), `jcabi-email`, `jcabi-jdbc` (`JdbcSession`), `jcabi-log` (`Logger.info(this, ...)`),
  `jcabi-aspects` (`@Immutable`, `@Cacheable`), `jcabi-dynamo` (`MkRegion`). Why: they expose
  interfaces, immutable implementations and built-in fakes. Example (Java):
  ```java
  String name = new JdbcSession(source)
    .sql("SELECT name FROM employee WHERE id = ?")
    .set(1234)
    .select(new SingleOutcome<String>(String.class));
  ```
  *(also Adt.68)*

- **Acoa.41 Know the multi-statement transaction pattern for `JdbcSession`** -> add to §11:
  `autocommit(false)` + `START TRANSACTION` … `commit()` keeps one connection open for several
  statements; by default the session closes after the first operation (single atomic transaction).
  Why: object-oriented JDBC hides SQL but still needs a transaction story. Example (Java):
  ```java
  new JdbcSession(source).autocommit(false)
    .sql("START TRANSACTION").update()
    .sql("DELETE FROM employee WHERE name = ?").set("Jeff Lebowski").update()
    .sql("INSERT INTO employee VALUES (?)").set("Walter Sobchak").insert(Outcome.VOID)
    .commit();
  ```

- **Acoa.42 Use XSLT 2.0 via Saxon and cache the stylesheet once** -> add to §11: Built-in Java XSLT
  supports only 1.0; add the Saxonica/Saxon runtime deps and build the `XSL` once (`XSLDocument.make`)
  as a private static, then `transform(xml)`. Why: creating a stylesheet per call is wasteful, and
  version 2.0 features (e.g. XPath 2.0) need Saxon. Example (Java):
  ```java
  private static final XSL STYLESHEET = XSLDocument.make(
    Foo.class.getResourceAsStream("stylesheet.xsl")
  );
  ```

- **Acob.45 Use Cactoos objects for declarative, lazy I/O** -> add to §11: compose `Input`/`Output`/`Bytes`/`Scalar` objects instead of calling `Files.readAllBytes`/`Files.write`; nothing touches the disk until a value is requested. Why: the file knows how to write itself (`FileAsOutput`), so adding an `OutputStream` sink is just a new secondary constructor, whereas the procedural `Encoder` would need ugly branching.
  ```java
  byte[] content = new InputAsBytes(new FileAsInput(new File("/tmp/photo.jpg"))).asBytes();
  new LengthOfInput(new TeeInput(input, new FileAsOutput(file))).value(); // writes here
  ```
  *(also Adt.64, Acs.37)*

- **Acob.46 xsline: immutable transformation pipeline with temporary generics** -> add to §11: use `Train.with(Shift)` / `Xsline.pass(doc)` for pipelines; add logging, classpath resolution, and bulk-adding as decorators (`TrLogged`, `TrClasspath`, `TrBulk`). Why: each concern is an isolated class, `with` keeps immutability, and `Train.Temporary.back()` returns to `Train<Shift>` without leaking a getter.
  ```java
  Train<Shift> train = new TrBulk<>(new TrClasspath<>(new TrDefault<>())).with(names).back();
  Document output = new Xsline(train).pass(input);
  ```

- **Atq.67 Adopt the EO Java testing toolbox** -> add to §11: Use JUnit 5 + Hamcrest (`assertThat`, `MatcherAssert`), fake objects, `jcabi-http` (`MkContainer`/`MkGrizzlyContainer`), Cactoos `Threads`/`RunsInThreads`, `jcabi-aspects` `@RetryOnFailure`, `jcabi-log`, and `jcabi-xml` (`XhtmlMatchers`). Why: these provide OO-friendly testing primitives that replace imperative scaffolding and (in tests) need no extra fixture code. Example: see Atq.32, Atq.38, Atq.27, Atq.16.

- **Atq.68 Use `XhtmlMatchers` for XML assertions** -> add to §11: Assert XML with `XhtmlMatchers.xhtml(...)` + `hasXPath("//foo")` rather than string or `notNullValue` checks. Why: it asserts actual structure and only prints the document when the assertion fails. Example (Java): see Atq.27.

- **Atq.69 Use JUnit 5 tags/annotations for test control** -> add to §11: Use `@Tag`, `@Disabled`, `@ExtendWith`, `@TempDir`, and `ParameterResolver` instead of home-grown categorisation. Why: native features cleanly separate fast/deep tests, park expected-failing tests, and inject prerequisites. Example (Java): see Atq.30/Atq.33/Atq.36.

- **Atq.70 Use the `tdx` tool to inspect test dynamics** -> add to §11: Analyse the ratio of test to production Hits-of-Code over the project lifecycle. Why: it empirically shows tests growing with reported bugs and supports the code→deploy→break→test→fix model. Example: run `tdx` on a repo and read the learning-curve-shaped graph.

- **Atq.71 Use `@RetryOnFailure` (jcabi-aspects) sparingly and explicitly** -> add to §11: Apply the annotation to idempotent, transient operations (JDBC SELECT, HTTP/S3/FTP download, REST GET). Why: it injects a bounded retry via AspectJ binary weaving without polluting the method body. Example (Java): see Atq.3/Atq.16.

- **Adt.59 Choose jcabi-http as the fluent, immutable HTTP client** -> add to §11: Use `new JdkRequest(uri).header(...).fetch().as(RestResponse.class).assertStatus(...).body()` with `JdkRequest`/`ApacheRequest` interchangeable behind `Request`/`Response`. Why: fluent one-statement requests, pluggable implementations, XPath/JSON parsing built in, `@Immutable` interfaces.

- **Adt.60 Use jcabi-dynamo / jcabi-jdbc instead of procedural AWS SDK / raw JDBC** -> add to §11: Interact with DynamoDB via `Region`/`Table`/`Item` objects and with SQL via `JdbcSession`. Why: hides REST/SQL details, exposes interfaces that can be mocked, and fits the no-DAO, object-oriented persistence rule.

- **Adt.61 Use jcabi-xml for XML, Xembly for building it** -> add to §11: Parse with `new XMLDocument("<root>...")` and `xpath()`/`nodes()`, and build with Xembly directives (`XeAppend`, `XeDirectives`, `XeChain`, `XeStylesheet`). Why: no DI/framework, dependency-free wrapper over native DOM, and no string concatenation when generating XML.

- **Adt.62 Use jcabi-ssh with `Shell` decorators for remote commands** -> add to §11: Wrap JSch in `new SSH(host, port, user, key)` and compose `Shell.Safe`, `Shell.Verbose`, `Shell.Plain`. Why: a single `exec` interface plus decorators replaces ad-hoc Process/SSH code and centralizes failure/logging.

- **Adt.63 Use jcabi-w3c and jcabi-matchers in tests** -> add to §11: Validate HTML/CSS with `ValidatorBuilder` and assert XML/XHTML with `XhtmlMatchers.hasXPaths(...)` (handles namespaces out of the box). Why: declarative, reusable test objects instead of procedural conversion boilerplate.

- **Adt.65 Use jcabi-maven-plugin to weave aspects** -> add to §11: When AspectJ weaving is unavoidable, add `jcabi-aspects` + `aspectjrt` and configure `jcabi-maven-plugin` with the `ajc` goal rather than hand-configuring the weaver. Why: it hides the complex weaving setup; note EO still prefers explicit decorators over aspects.

- **Adt.66 Use Takes (Tk/Rq/Rs/Fk/Ts) to build a web app object-orientedly** -> add to §11: Implement `Take.route`/`act`, decorate `Request`/`Response`, route with `TkFork`+`FkRegex`/`FkParams`, and serve via `FtBasic`/`FtCli`. Why: no statics, no NULL, no mutable classes, no casting — every take is a small immutable, independently testable object. Example:
  ```java
  final class TkApp extends TkWrap {
    TkApp() { super(new TkFork(new FkRegex("/", new TkIndex()))); }
  }
  ```

- **Adt.67 Serve XML data + XSL views instead of a templating engine** -> add to §11: Point a `RsXSLT` response at `<?xml-stylesheet href='/xsl/index.xsl'?>` and select the XSL transform server-side when the client cannot do XSLT. Why: one URL serves both the API (XML) and the site (HTML), removing duplicated controllers; the XML is testable with XPath and views are stackable/reusable.

- **Adt.69 Wrap command-line-only tools (Nutch) via download plugin + config** -> add to §11: For tools distributed as binaries, depend on the Maven artifact, use `download-maven-plugin:wget` to fetch and unpack the distribution, set mandatory config (`http.agent.name`, `plugin.folders`), and copy `conf/` into `src/main/resources`. Why: lets a CLI-only framework be driven as a library from Java, reproducibly. Example:
  ```xml
  <plugin>
    <groupId>com.googlecode.maven-download-plugin</groupId>
    <artifactId>download-maven-plugin</artifactId>
    <version>1.4.1</version>
    <executions><execution><phase>generate-resources</phase><goals><goal>wget</goal></goals>
      <configuration>
        <url>http://artfiles.org/apache.org/nutch/1.15/apache-nutch-1.15-bin.zip</url>
        <unpack>true</unpack>
        <outputDirectory>${project.build.directory}</outputDirectory>
      </configuration>
    </execution></executions>
  </plugin>
  ```

- **Adt.70 Filter an external notification firehose to stay productive** -> add to §11: Register notification sources as "pipes" and filter them by text/regex (e.g. wring.io with `AgGithub`). Why: hundreds of GitHub emails per day destroy focus; a configurable dispatcher keeps only actionable notifications.

- **Ada.37 Use Xembly, not JAXB, for Java-to-XML** -> add to §11: Marshal objects by returning `Iterable<Directive>` and rendering with `Xembler`; do not annotate getters for JAXB. Why: JAXB treats the object as a passive data bag it can mine freely, whereas Xembly keeps the object in control and avoids getters and XML annotations. Example (Java):
  ```java
  final Book book = new Book("0132350882", "Clean Code");
  final String xml = new Xembler(book.toXembly()).xml();
  ```
  *(also Acoa.43)*

- **Ada.38 Use Cactoos scalars for lazy, cached, checked values** -> add to §11: Reach for `Scalar`, `StickyScalar`, `SyncScalar` and `IoCheckedScalar` instead of hand-rolled null/mutable lazy fields. Why: they compose as decorators to give lazy loading, caching and thread-safety without breaking immutability. Example (Java):
  ```java
  this.text = new IoCheckedScalar<>(
    new SyncScalar<>(new StickyScalar<>(source))
  );
  ```

- **Ada.39 Model an HTTP server with `Resource`/`Output`, not a web framework** -> add to §11: Build web handling from `Resource.refine()` (immutable request mutation) and `Output.print()` (self-rendering response) instead of installing Spring with DTO controllers. Why: a DTO-free resource that is both data and processor dispatches requests in a purely object-oriented way, testable without a container. Example (Java):
  ```java
  interface Resource {
    Resource refine(String name, String value);
    void print(Output output);
  }
  Resource r = new DefaultResource()
    .refine("X-Method", "GET")
    .refine("X-Query", "/");
  ```

- **Ada.40 Avoid DI frameworks (Guice/Spring/Dagger) entirely** -> add to §11: Do not add a DI container; express composition with constructors and `new`. Why: containers pollute code with annotations and configuration files while adding only complexity, not behavior. Example (Java):
  ```java
  // Instead of @Inject + Guice.createInjector(...):
  final Budget budget = new Budget(new Postgres(dsn));
  ```

- **Acs.36 Use manifest-reading and build-number tooling** -> add to §11: Read package attributes with jcabi-manifests (`Manifests.read`), and generate the VCS hash with buildnumber-maven-plugin. Why: multiple JARs all carry `META-INF/MANIFEST.MF`; a one-liner reader plus the Maven plugin supplies fresh version/hash attributes without hand-rolled classpath scanning. Example (Java):
  ```java
  import com.jcabi.manifests.Manifests;
  String created = Manifests.read("Created-By");
  ```

- **Acs.38 Do not use the Java Validation API** -> add to §11: Avoid annotation-based validation (`javax.validation`) because its annotations make classes verbose and less cohesive; use validating decorators instead. Why: bean-validation annotations spread validation metadata across the class and tie it to a framework, while decorators keep each validation in one small reusable object. Example (Java):
  ```java
  final class NoNullReport implements Report { /* decorator instead of @NotNull fields */ }
  ```

- **Apa.99 Automate PDD with the pdd gem and 0pdd** -> add to §11: Scan the codebase for `@todo` markers, open/close GitHub issues automatically, and show a puzzle badge. Why: the toolchain turns puzzles into a managed backlog without a manager. Example:
  ```text
  gem install pdd / hosted 0pdd bot  +  pdd badge on the README
  ```
  *(also Apb.68)*

- **Apa.100 Use Requs for machine-checkable SRS** -> add to §11: Write `.req` files under `src/main/requs`, compile them in CI, view the generated SRS. Why: controlled natural language makes requirements parseable and incrementally editable. Example:
  ```text
  mvn clean requs:compile   → target/requs/index.xml
  ```

- **Apa.101 Use Hits-of-Code (`hoc`) for effort metrics** -> add to §11: Install the `hoc` gem/site and report HoC alongside LoC. Why: it measures actual churn/effort, not snapshot size. Example:
  ```text
  gem install hoc; hoc → 54687;  https://hitsofcode.com/
  ```

- **Apa.102 Use Rultor as the merge/release bot** -> add to §11: Drive merge and release from ticket comments; the bot tests, merges, packages and deploys. Why: one deterministic bot replaces ad-hoc human release steps. Example:
  ```text
  @rultor merge   /   @rultor release, tag is v1.2.3
  ```
  *(also Apb.66)*

- **Apa.103 Use Qulice (and equivalents) for mandatory Java static analysis** -> add to §11: Run Qulice in the build and fail on any violation; use RuboCop for Ruby. Why: it is the concrete enforcement mechanism behind the mandatory-static-analysis rule. Example:
  ```text
  qulice.com in the Maven build → build fails on any rule violation
  ```
  *(also Apb.69, Acob.47)*

- **Apa.104 Validate rendered HTML in tests with PhantomJS/Phandom** -> add to §11: Feed generated HTML to a headless WebKit and assert the DOM, skipping when the tool is absent. Why: catches broken markup/JS that unit tests miss. Example:
  ```java
  Assume.assumeTrue(Phandom.installed());
  assertThat(XhtmlMatchers.xhtml(new Phandom(html).dom()),
             XhtmlMatchers.hasXPath("//p[.='Hello, world!']"));
  ```

- **Apa.105 Use chat-as-interface for microservices instead of a web UI** -> add to §11: Make the service a client of a communication hub (GitHub/Netbout), talking and being talked to via messages. Why: no UI to design, natural tolerance for latency, easy scaling and traceability of the whole conversation. Example:
  ```text
  User posts to ticket → bot reads via API → replies in ticket → user gets a notification.
  ```

- **Apa.106 Use Markdown in the repo for technical documents and diagrams** -> add to §11: Keep decisions, schemas, glossaries and vision in version-controlled Markdown under Git. Why: diffable, reviewable via PRs, and permanently searchable. Example:
  ```text
  README.md, schema.md, vision.md  →  reviewed as pull requests
  ```

- **Apa.107 Reference tickets in commit messages so the tracker auto-links** -> add to §11: Start each commit message with the issue number; GitHub links commit↔ticket automatically. Why: it makes every change traceable to what/who/why in seconds. Example:
  ```text
  "#123 add DataBridge interface"
  ```

- **Apb.65 Plain-text config mirrors CLI flags** -> add to §11: Store tool defaults in a plain-text
  file with one CLI flag per line (e.g. `.xcop`, `~/.xcop`) and concatenate file + command-line args;
  do not introduce YAML/JSON/TOML for the same options. Why: users learn one format only, and the same
  syntax works interactively and in config. Example:
  ```text
  # .xcop
  --include=*.xml
  --exclude=.idea/**
  --license=LICENSE.txt
  $ xcop   # picks up the file automatically
  ```

- **Apb.67 Managed cloud Maven repo beats raw S3** -> add to §11: Prefer a managed repository manager
  (e.g. CloudRepo) over a hand-rolled S3 bucket when you need users/groups, Maven views, webhooks,
  audits and managed security. Why: a repository manager is "a Maven repository in cloud," not just a
  JAR store, and removes IAM/S3 administration. Example:
  ```text
  settings.xml <server id=io.cloudrepo> + pom <distributionManagement> -> @rultor deploy
  ```

- **Apb.70 Let AI run the mechanical repository chores** -> add to §11: Delegate bug reporting, PR
  review, small refactors, backlog prioritization, documentation sync and estimation to AI agents,
  keeping each automated PR small and easily mergeable. Why: these are the routine, boring tasks humans
  skip; robots "write reports so nicely" that programmers prefer them, and small incremental PRs keep
  trust. Example:
  ```text
  agent -> reword bug report with reproducer + stack trace
  agent -> open tiny refactor PR, one concern, easy to merge
  ```

### Additions from real codebases

- **Zero runtime dependencies is the default for a library.** Cactoos README: *"The library has no
  dependencies"* / *"Zero runtime dependencies"* (`README.md:32,414-419`); xembly's only compile deps
  are Lombok + xml-apis, both `provided` (`pom.xml:69-80`). Takes' POM policy comment: *"we are NOT
  allowed to use ANY third-party dependencies here, unless they are in PROVIDED, TEST or RUNTIME
  scope. The project must be self-sufficient and work in a single JAR dependency."*
- **`cactoos` is the one universal runtime dependency** where an app needs OO primitives: Takes
  `org.cactoos:cactoos:0.63.0`, s3auth `0.61.1`, rehttp, jare `0.61.0`, Qulice `0.62.0`, eo. Cactoos
  itself excludes self-dependency from `takes`/`cactoos-matchers` test deps (`pom.xml:74-79,86-91`).
- **jcabi-* family replaces Guava/Apache Commons utilities.** `jcabi-http`
  (`Request`/`Response` fluent + `MkContainer`), `jcabi-xml` (`XMLDocument`, `XhtmlMatchers`),
  `jcabi-aspects` (`@Immutable`, `@Cacheable`, `@Loggable`, `@Timeable`, `@RetryOnFailure`,
  `@RetryOnFailure`), `jcabi-jdbc` (`JdbcSession`), `jcabi-dynamo`, `jcabi-s3` (`Ocket`),
  `jcabi-manifests` (`Manifests.read`), `jcabi-ssh` (`Shell` decorators), `jcabi-log`,
  `jcabi-matchers`.
- **OO XML tooling.** Xembly 0.32.2 for building XML from `Iterable<Directive>` (Takes `RsXembly`,
  s3auth, rultor, requs); Saxon-HE for XSLT 2.0/XPath 2.0 (built-in Java XSLT is 1.0 only).
  `jcabi-xml` for parsing. XML/XSL views avoided a templating engine in Takes (`RsXSLT`).
- **Web framework is Takes** (`org.takes:takes`): `Tk`/`Rq`/`Rs`/`Fk`/`FtRemote`; deployments use
  fat-classpath `target/app.jar:target/deps/*` from a `heroku`/`dokku` profile and Takes CLI flags
  `--port`, `--threads`, `--max-latency` (rultor/jare `Procfile`).
- **Persistence behind interfaces; ORM/SQL is a detail.** rehttp DynamoDB behind `Base`/`Status` with
  AWS types only inside `Dy*` (`DyBase.java:7-16`); requs JAXB/XML via Xembly; rultor/jare AWS SDK
  (v2 in rultor, v1 in jare) hidden in `dynamo/` classes.
- **Testing toolbox.** JUnit 5 + Hamcrest `MatcherAssert.assertThat(reason, …)`; `cactoos-matchers`;
  `jcabi-matchers`; `MkGrizzlyContainer`/`MkContainer` for HTTP; `jqwik` property tests; JMH; JUnit
  Pioneer for env/temp injection; `rerunner-jupiter`; `together`/`farea`/`mktmp`/`jhome` test
  harnesses; `DynamoDBLocal` + `build-helper:reserve-network-port` for ITs.
- **Build/infra plugins worth cataloguing.** `revapi` (API compatibility), `pitest`, `jacoco`,
  `forbiddenapis`, `spotbugs`, `proguard` (xembly), `maven-verifier`, `flatten-maven-plugin`,
  `license-maven-plugin`, `maven-invoker-plugin`, `download-maven-plugin`/`exec-maven-plugin` (fetch &
  run CLI tools), `sass-maven-plugin`, `maven-assembly-plugin`, `hone-maven-plugin` (EOLANG;
  `skipOnWindows`/`skipWithoutDocker`), `nexus-staging-maven-plugin` (Central), `maven-s3-wagon`.
- **Classpath hygiene by exclusion.** Qulice strips ASM/Gson/checker-qual from PMD/ErrorProne to
  avoid clashes with its own `org.ow2.asm:asm:9.10.1` (`pom.xml:287-337`); `takes` excludes transitive
  `cactoos`. Rule: pin versions in `dependencyManagement`/BOM (s3auth AWS BOM 2.44.4; eo `junit-bom`)
  and exclude transitive duplicates.

## 12. Recurring problems and their solutions

- **B12.1 Repeated literal → micro-class** — Same `"\r\n"` (or `"POST"`) in many places must become a
  class (`CRLFString`, `PostRequest`), not a `public static final` or `enum`. (B1.15, B4.12, B8.7)
  Example (Java):
  ```java
  class CRLFString { private final String origin; CRLFString(String s) { this.origin = s; } @Override public String toString() { return String.format("%s\r\n", origin); } }
  ```

- **B12.2 Static utility call → object wrapper** — Replace `Math.max(5,9)` / `FileUtils.readLines()`
  with `new Max(5, 9)` / `new FileLines(f)`, confining the static call to one class. (B1.9, B1.10,
  3.2.1, B5.11)
  Example (Java):
  ```java
  int x = new Max(5, 9).intValue();     // declarative
  Iterable<String> lines = new FileLines(f);
  ```

- **B12.3 Hard-coded dependency → constructor injection** — Replace `new Exchange()` inside a method
  with an injected field initialized in the primary ctor. (B1.19, B2.9, B3.6/3.6)
  Example (Java):
  ```java
  final class Cash {
      private final int dollars; private final Exchange exchange;
      Cash(int value, Exchange exch) { this.dollars = value; this.exchange = exch; }
      int euro() { return (int) (this.exchange.rate("USD", "EUR") * this.dollars); }
  }
  ```

- **B12.4 Null argument → Null Object** — Replace `find(String mask == null)` with a `Mask` interface
  and an `AnyFile` implementation. (B6.1)
  Example (Java):
  ```java
  interface Mask { boolean matches(File file); }
  class AnyFile implements Mask { @Override public boolean matches(File file) { return true; } }
  ```

- **B12.5 Null return → throw / empty collection / Null Object** — Replace `return null` with one of
  the three alternatives. (B6.3)
  Example (Java):
  ```java
  Collection<User> users(String name) { return java.util.Collections.emptyList(); }
  ```

- **B12.6 Mutable bean → immutable constructed object** — Replace no-arg ctor + setters with a
  complete ctor and `final` fields. (B2.15, B3.1)
  Example (Java):
  ```java
  // before: new Cash(); setDollars(29); setCents(95);
  // after:
  Cash price = new Cash(29, 95);
  ```

- **B12.7 Mocking → fake class** — Replace Mockito stubs with a nested `Fake`. (B7.4, B7.5)
  Example (Java):
  ```java
  Exchange exchange = new Exchange.Fake(1.2345f);
  ```

- **B12.8 Mutable lazy field → caching decorator** — Replace `null`-sentinel lazy loading with a
  `CachedNumber`/decorator wrapper. (B2.8, B3.10)
  Example (Java):
  ```java
  Number num = new CachedNumber(new StringAsInteger("123"));
  ```

- **B12.9 `instanceof`/cast → overloading or polymorphism** — Replace runtime type checks with
  overloaded methods or decorators. (B1.14, 3.7)
  Example (Java):
  ```java
  public <T> int size(java.util.Collection<T> items) { return items.size(); }
  public <T> int size(Iterable<T> items) { int n = 0; for (T i : items) { ++n; } return n; }
  ```

- **B12.10 Getters/setters → behavior methods** — Replace `getDollars()`/`setDollars()` with
  `dollars()` / a behavior such as `add(Cash)`. (B1.13, B4.11, B4.13)
  Example (Java):
  ```java
  class Cash { private final int value; public int dollars() { return this.value; } }
  ```

- **B12.11 `-er` class name → entity name** — Rename `FileReader` → `DataFile`, `CashFormatter` →
  `Cash`, `PrimeFinder` → `PrimeNumbers`. (B1.12, B4.1, B4.2)
  Example (Java):
  ```java
  final class EncodedText { private final String value; EncodedText(String v) { this.value = v; } String text() { return this.value; } }
  ```

- **B12.12 Long class → split into cohesive micro-classes** — If LOC > 250 or public methods ≥ 5,
  extract classes; if state > 4 coordinates, group into sub-objects. (B1.7, B1.20, B1.21, B8.1)
  Example (Java):
  ```java
  // 1000-line class -> several final classes, each with <5 public methods and <250 LOC.
  ```

- **B12.13 Boolean/query method with verb name → adjective** — Rename `exists()` → `present()`,
  `isEmpty()` → `empty()`. (B4.5)
  Example (Java):
  ```java
  boolean present(); boolean empty(); boolean readable();
  ```

- **B12.14 Builder → small immutable collaborators** — Stop assembling a giant object with `withX()`
  chains; pass small objects to the ctor instead. (B5.8)
  Example (Java):
  ```java
  new Book(new Author("A"), new Title("T"), new Page(page));
  ```

- **B12.15 Interface bloat → short interface + nested `Smart`** — Split convenience methods out of the
  contract into `Interface.Smart`. (B4.9, B4.10)
  Example (Java):
  ```java
  interface Exchange {
      float rate(String source, String target);
      final class Smart { private final Exchange origin; Smart(Exchange e) { this.origin = e; } float eurToUsd() { return this.origin.rate("EUR", "USD"); } }
  }
  ```

- **B12.16 Early catch/recover → chain & rethrow, recover once at top** — Remove mid-stack recovery;
  wrap and rethrow; keep one catch in `main`. (B6.6, B6.7, B6.8)
  Example (Java):
  ```java
  try { return content(file).length; } catch (IOException ex) { throw new Exception("Can't calculate length.", ex); }
  ```

- **B12.17 Catch-and-log → rethrow** — Never swallow an exception into the log. (B6.10)
  Example (Java):
  ```java
  // before: catch (IOException ex) { log.error("oops", ex); }
  // after:  catch (IOException ex) { throw new Exception("context", ex); }
  ```

- **B12.18 Sentinel `-1`/`0` → exception, collection, or Null Object** — Stop encoding failure as a
  scalar. (B6.12)
  Example (Java):
  ```java
  int length(File file) throws IOException { return content(file).length; }
  ```

- **B12.19 Inheritance for reuse → interface + decoration** — Make the base final, extract an
  interface, wrap. (B5.1, B5.3, B5.4)
  Example (Java):
  ```java
  final class EncryptedDocument implements Document { private final Document plain; EncryptedDocument(Document d) { this.plain = d; } @Override public byte[] content() { return decrypt(this.plain.content()); } }
  ```

- **B12.20 Imperative loop/if → declarative object** — Replace `for`/`if` with `Filtered`/`If`
  objects. (B1.17, B1.24, B5.9, B8.6)
  Example (Java):
  ```java
  Collection<Integer> evens = new Filtered<>(numbers, number -> number % 2 == 0);
  float rate = new If(new GreaterThan(new AgeOf(client), 65), 2.5f, 3.0f);
  ```

- **B12.21 Constructor with logic → code-free wrapper + deferred method** — Move parsing/validation
  out of ctors. (B1.3, B2.1, B2.5)
  Example (Java):
  ```java
  class StringAsInteger implements Number { private final String source; StringAsInteger(String s) { this.source = s; } public int intValue() { return Integer.parseInt(this.source); } }
  ```

- **B12.22 Too many ctor arguments → group into sub-objects** — A growing arg list is a signal to
  extract collaborator objects. (B1.7, B2.6, B12.12)
  Example (Java):
  ```java
  final class Cash { private final Digits digits; private final Currency currency; /* ... */ }
  ```

- **B12.23 Global state / singleton → injection** — Encapsulate every shared dependency (current
  user, session, storage) in each object via its ctor. (B1.11)
  Example (Java):
  ```java
  final class Report { private final User user; Report(User u) { this.user = u; } }
  ```


### Additions from articles

- **Acoa.44 Replace ORM with "SQL-speaking objects" (singular + plural classes)** -> add to §12:
  Model the table and the row as two objects (`Posts`, `Post`) implementing interfaces; each hides
  its SQL and can be decorated/optimised (`ConstPost` caches row data fetched in one round trip).
  Why: ORM tears one cohesive object into a DTO + a session engine, exposing SQL and making tests
  slow. Example (Java):
  ```java
  interface Posts { Iterable<Post> iterate(); Post add(Date date, String title); }
  interface Post  { int id(); Date date(); String title(); }
  ```
  *(also Acob.48, Acoa.46)*

- **Acoa.45 Encapsulate transactions per object, with a `Txn` callable wrapper if needed** -> add to
  §12: Let each object own its transaction (nested transactions are fine); when unsupported, wrap a
  group of manipulations in `new Txn(dbase).call(...)`. Why: keeps transaction details behind the
  objects that need them instead of leaking a session into every caller. Example (Java):
  ```java
  new Txn(dbase).call(() -> {
    Posts posts = new PgPosts(dbase);
    Post post = posts.add(new Date(), "How to cook an omelette");
    post.comments().post("This is my first comment!");
    return post.id();
  });
  ```

- **Acoa.47 Don't make object behavior configurable — inject behavior via decorators** -> add to §12:
  A growing constructor with `encoding`, `alwaysHtml`, `encodeAnyway` (and a `PageSettings` holder)
  should become `NeverEmptyPage(TextPage(DefaultPage(url), enc))` etc. Why: configuration flags as
  properties make classes big, non-cohesive and untestable; properties may only locate the entity,
  not change behavior. Example (Java):
  ```java
  Page page = new NeverEmptyPage(
    new OncePage(new DefaultPage("https://www.google.com"))
  );
  String html = new AlwaysTextPage(new TextPage(page, "ISO_8859_1"), page).html();
  ```

- **Acoa.48 Turn every singleton into a constructor dependency** -> add to §12: Pass a `Database`
  instance to all objects that need it through constructors; if that means 10 arguments, split the
  object instead. Why: singletons are global state; `Database.INSTANCE` in a REST controller is
  untestable and couples everything. Example (Java):
  ```java
  // not Database.INSTANCE.connect(), but:
  final class Index { private final Database db; Index(Database d) { this.db = d; } }
  ```

- **Acoa.49 Kill DTOs by keeping data inside the object** -> add to §12: Replace
  `api.loadBookById(id)` + `database.saveNewBook(book)` (DTO passing) with `api.bookById(id).save(db)`;
  data never escapes the object. Why: DTO is a passive anemic box that turns OO into procedures.
  Example (Java):
  ```java
  Book book = api.bookById(123);
  book.save(database);
  ```

- **Acoa.50 Purge utility classes in favour of small collaborating objects** -> add to §12: Replace
  `FileUtils.readLines/writeLines` with objects: `Trimmed`, `FileLines`, `UnicodeFile` all implement
  `Collection<String>`; laziness and O(1) space follow. Why: a 3000-line `FileUtils` is hard to
  maintain/test; small classes each own one feature. Example (Java):
  ```java
  Collection<String> src = new Trimmed(new FileLines(new UnicodeFile(in)));
  Collection<String> dest = new FileLines(new UnicodeFile(out));
  dest.addAll(src);
  ```
  *(also Adt.73)*

- **Acoa.51 Make logic lazy/declarative to avoid unnecessary work** -> add to §12: A composed object
  (`Postman`/`Envelope`/`Stamps`) only touches I/O when a terminal method (`send`, `read`) is called;
  procedural code computes "right here and now". Why: laziness follows naturally from composition
  and avoids work when a later step fails. No new snippet; cross-ref §5.

- **Acob.49 Do not fear a large number of small classes** -> add to §12: when rules produce hundreds of classes, keep them small, noun-named, prefixed, with one-word methods; treat the count as a sign of a rich vocabulary, not a defect. Why: the author reports Takes at 24k lines / 410 files / 10 prefixes, with the longest class name `RqWithDefaultHeader`; larger, low-cohesion classes are harder to read and maintain. Prefixes keep names short and understandable.
  ```java
  // Takes prefixes: Bc, Cc, Tk, Rq, Rs, Fb, Fk, Hm, Ps, Xe
  final class RqWithHeader implements Request { /* ... */ }
  ```

- **Atq.72 "We have no time for tests" is a skill gap, not a schedule problem** -> add to §12: Treat refusal to write tests as not knowing how; a unit test is a tool that makes development faster, like a class. Why: code without tests is as absurd as a 20,000-line single-class program. Example: respond to "no time for tests" with "then how did you verify the code works?".

- **Atq.73 "Tests can never finish"** -> add to §12: Because defects are unlimited, use a forecast bug count as exit criteria and release knowing bugs remain. Why: release happens when the error-discovery rate drops to a management-acceptable level, not at zero defects. Example: West: "software is released … when the rate of discovering errors slows down to one that management considers acceptable".

- **Atq.74 "A patch fixes one thing and breaks others"** -> add to §12: Reject fixes that reduce coverage, fail to reproduce, or are too broad; require a reproduction test and one issue per PR. Why: merging such patches conceals missing coverage and increases entropy. Example: a PR that deletes or `@Disabled`s failing tests without new tests is rejected.

- **Atq.75 "The bug is trivial — just remove the typo"** -> add to §12: The real problem is the absent test, not the typo; every fix must include a test that catches the typo at deploy time. Why: fixing the symptom without coverage guarantees recurrence. Example: add the test first, watch it fail, then fix.

- **Atq.76 "Debugging is faster than writing a test"** -> add to §12: If writing the test is expensive, that signals a design problem; refactor into smaller noun-objects until the test is trivial. Why: hard-to-test code is intrinsically hard to maintain, and debugging wastes energy without preventing recurrence. Example: split `FileUtils.readWords(File)` into `Text` (file→string) and `Words` (string→iterable).

- **Atq.77 "Refactor while fixing" temptation** -> add to §12: Separate refactoring from bug fixes into its own paid ticket/PR. Why: refactoring is a design bug that deserves its own review and traceability, and mixing it obscures the fix. Example: after the minimal fix, open "simplify `Foo`".

- **Atq.78 "One giant catch block is convenient"** -> add to §12: Split into one small `try`/`catch` per originator with a specific message. Why: a shared catch cannot report which call failed; per-originator context makes the failure debuggable. Example: see Atq.12/Atq.13.

- **Atq.79 "ConcurrentHashMap means thread-safe"** -> add to §12: Any compound check-then-act must be synchronized and covered by a latch-driven parallel test. Why: field-level thread safety does not protect multi-step invariants. Example: see Atq.4/Atq.31.

- **Atq.80 "Our tests are slow, so delete the slow ones"** -> add to §12: Split fast vs deep and run deep tests on the server, not locally; never delete them. Why: deep tests catch leaks and resource limits that fast/mocked tests by design cannot. Example: see Atq.30 and the open-file-limit test.

- **Adt.71 Don't concatenate strings — format or join** -> add to §12: Replace `+` concatenation with `String.format` (with argument indexes for localizability) or a join, especially for long multi-line text. Why: concatenation is less readable and not localizable; the readability cost exceeds any performance gain. Example:
  ```java
  String msg = String.format(
    "Dear %1$s, your order #%2$d has been shipped at %3$tR!",
    customer.name(), order.number(), shipment.date()
  );
  ```

- **Adt.72 Prevent resource leaks with explicit acquisition/release** -> add to §12: On every exception path that leaves a method, ensure the acquired resource is released before the `throw`, or wrap it in a `Closeable` used with try-with-resources. Why: an unreleased permit/connection is exhausted after a few failures and blocks all threads.

- **Adt.74 Deduplicate repetitive build configuration with a parent POM** -> add to §12: Put common plugins/dependencies/versions in `com.jcabi:parent` (or a project parent) and inherit it, rather than copying 100+ lines into every `pom.xml`. Why: Maven verbosity is its chief weakness; a shared parent removes version drift and duplicated plugin config. Example:
  ```xml
  <parent>
    <groupId>com.jcabi</groupId>
    <artifactId>parent</artifactId>
    <version>0.32.1</version>
  </parent>
  ```

- **Adt.75 Interrupt-based time limits only work if the method cooperates** -> add to §12: When limiting execution time (`@Timeable(limit=5, unit=SECONDS)`), make loops check `Thread.interrupted()` and throw `InterruptedException`; never rely on `Thread.stop`. Why: `interrupt()` only sets a flag; a thread that ignores it (or a tight `while(true)` loop) cannot be stopped except by killing the JVM.

- **Adt.76 Avoid defensive catch-and-continue and exception-driven flow control** -> add to §12: Don't translate a caught exception into a sentinel default; propagate it and let the caller decide. Why: silent defaults mask real faults and produce wrong behavior (e.g. a full disk reported as an empty file).

- **Adt.77 Don't let annotations/configurations push behavior out of objects** -> add to §12: The recurring fix for `@Inject`, JAXB, `@RetryOnFailure`, Spring XML, etc. is composition: inject dependencies via constructors and add behavior with decorators. Why: every configuration/annotation mechanism tears an object apart and hides the instantiation/composition that must be visible.

- **Adt.78 Prefer immutable, interface-typed, no-null objects across tool boundaries** -> add to §12: When integrating any library, wrap it in your own small interface implemented by immutable classes, avoiding null/static/casts. Why: the same four principles (no NULL, no statics, no mutability, no casting/instanceof/reflection) applied to third-party code keep architecture clean and testable.

- **Ada.41 Replace getter-based data extraction with printers** -> add to §12: When you feel tempted to add getters so a tool can read an object, instead let the object print itself to a sink. Why: getters exist only to export data a framework wants; printers preserve encapsulation and keep representation logic inside the object. Example (Java):
  ```java
  void print(final Output output) {
    output.print("Content-Type", "text/plain");
    output.print("X-Body", this.body);
  }
  ```

- **Ada.42 Fix "lazy loading is ugly" with a scalar, not a nullable field** -> add to §12: The recurring problem of lazy loading wanting a null field is solved by encapsulating a function and decorators. Why: a `Scalar` field is final and null-free, and sticky/sync decorators add caching and thread-safety without mutation. Example (Java):
  ```java
  Encrypted4(final Scalar<String> source) {
    this.text = new IoCheckedScalar<>(source);
  }
  ```

- **Ada.43 Treat "responsibility" as encapsulation, not as a count** -> add to §12: When reviewing a class that "does too much", do not split it into getters+checkers+readers+writers; ask whether it hides its dependencies. Why: the SRP refactoring turns a working object into a carrier of a client, wrecking decorability and encapsulation. Example (Java):
  ```java
  // Keep: ocket.exists(), ocket.read(out) — the object owns its AWS client.
  // Not: new ExistenceChecker(ocket.aws()).exists()
  ```

- **Ada.44 Resolve dependency conflicts with the trust-based version rule** -> add to §12: When two libraries demand different versions of a shared dependency, decide dynamic-vs-fixed by how much you trust the upstream's semantic versioning. Why: Maven can resolve conflicts but the library author can still break you silently; the hybrid rule trades conflict risk against time-bomb risk. Example (XML):
  ```xml
  <dependency>
    <groupId>com.example</groupId>
    <artifactId>log-me</artifactId>
    <version>1.13.5</version>
  </dependency>
  ```

- **Ada.45 Keep PostgreSQL context where the data is born** -> add to §12: When choosing between fat and skinny designs, do the complex data parsing inside the class that talks to the database; do not push connection details up a chain of adapters. Why: losing PostgreSQL context far from `PgArticle` is a bigger encapsulation violation than letting raw text escape; prefer skinning until it would leak driver-specific data. Example (Java):
  ```java
  interface Article { String head(); }
  final class SqlArticle implements Article { /* owns the SQL + parsing */ }
  ```

- **Acs.39 Empty line signals a multi-responsibility method** -> add to §12: When you see a blank line inside a method (or a blank-line-separated CSS block), refactor the method/class into cohesive pieces. Why: the whitespace marks where the author mentally split unrelated concerns; converting that split into real methods/classes restores single responsibility (cf. Acs.24). Example (Java):
  ```java
  public int grep(Pattern regex) throws IOException {
    return this.count(this.lines(), regex);
  }
  ```

- **Acs.40 Two-step initialization masks a design flaw** -> add to §12: Treat any `init()`/`setup()`/`open()` requirement as a red flag and refactor toward immutable, fully-constructed objects. Why: it hides mutability, fragile base classes, oversized abstractions, and circular layering; fixing the design removes the need for the second step (cf. Acs.4). Example (Java):
  ```java
  final class Book implements Closeable {
    private final InputStream in;
    Book(InputStream stream) { this.in = stream; }   // no init()
  }
  ```

- **Acs.41 Inline validation hides a missing decorator** -> add to §12: When a method fills with null/exists checks, extract them into validating decorators. Why: defensive inline checks bloat the core class and mix concerns; decorators restore smallness and reuse (cf. Acs.16, Acs.21). Example (Java):
  ```java
  Report report = new NoNullReport(
    new NoWriteOverReport(new DefaultReport())
  );
  ```

- **Acs.42 Getter + setter is naked data in disguise** -> add to §12: When a class is just private fields with getters/setters, replace the accessors with behavior. Why: it looks encapsulated but still lets callers read and reinterpret raw data, preserving hidden coupling and blocking polymorphic behavior (cf. Acs.1, Acs.9). Example (Java):
  ```java
  final class Temperature {
    private final int t;
    Temperature(int value) { this.t = value; }
    public String toString() { return String.format("%d F", this.t); }
  }
  ```

- **Apa.108 Delegating questions to the manager reattaches the "monkey"** -> add to §12: When a teammate says "I don't know what to do", push the problem back as their task, or ask for a ticket that pays for the answer. Why: excuses transfer responsibility; the manager's job is to keep the monkey on the performer's shoulders. Example:
  ```text
  Programmer: "I don't know how."  Manager: "File a ticket about the missing docs."
  ```

- **Apa.109 Volunteer quality tools create an illusion of quality** -> add to §12: Any quality mechanism not wired into a failing gate will be ignored within weeks. Why: teams under deadline filter out alerts; only a red build changes behavior. Example:
  ```text
  Jenkins alerts → filtered to a folder → ignored. Fix: merge gate rejects the branch.
  ```

- **Apa.110 Long-running tickets make a developer unmanageable** -> add to §12: Deliver quickly (even by disabling a feature) rather than holding a ticket for days. Why: a held ticket cannot be forecast or reassigned; speed keeps the workflow legible. Example:
  ```text
  "Production errors are not programmers' mistakes, but delayed tickets are."
  ```

- **Apa.111 Hidden source code destroys trust; radical transparency keeps customers** -> add to §12: Give sponsors day-one access to repo, CI, tickets and discussions; formalize and bill their questions. Why: hiding sources signals weak quality; visibility plus independent reviews builds durable trust. Example:
  ```text
  Client question → must become a ticket → answered through the full flow → billed.
  Client demand → must amend requirements, never direct-to-code instructions.
  ```

- **Apa.112 Runaway client demands are neutralized by requirements, not argument** -> add to §12: Convert every "use MySQL" demand into a requirements change; non-sane demands die in the process. Why: it is a win-win that protects the team from micromanagement without insulting the client. Example:
  ```text
  Demand: "Use MySQL."  Requirement attempt: "Use only great databases."
  (PostgreSQL qualifies; the demand dissolves.)
  ```

- **Apa.113 Beware the false objective of a happy boss/customer** -> add to §12: Optimize the process and the artifact, not anyone's feelings; satisfaction is a consequence. Why: chasing happiness invites pleasing, corner-cutting and hiding problems. Example:
  ```text
  "Don't ask clients what they want — learn their business, then recommend and defend the best solution."
  ```

- **Apa.114 Compromise is the worst conflict outcome; seek the underlying interest** -> add to §12: Replace positions with interests and find a solution satisfying all, or let the architect decide—never split the difference. Why: compromise satisfies nobody and leaks accountability; win-win exposes information. Example:
  ```text
  Position: "I want the movie." → Interest: "I'm tired."
  → "I watch the game and give you a massage."
  ```

- **Apa.115 Resolve code-review conflicts in exactly three ways** -> add to §12: Accept you were wrong, hold firm, or appeal to the architect; never compromise. Why: compromise ruins quality faster than bad code and masks unresolved disagreement. Example:
  ```text
  "You're right; I take it back." / "I will never accept this." / "Let's do what the architect says."
  ```

- **Apa.116 A reviewer must prove the code is bad; the author must not prove it is good** -> add to §12: Ground every comment in links, articles, examples; "15 years of Java" is not proof. Why: the reviewer is plaintiff, the author defender; without evidence the reviewer may simply be wrong. Example:
  ```text
  Every rejection carries: reference + concrete counter-example + requested change.
  ```

- **Apa.117 Review without fear, compromise, bullshit or offense** -> add to §12: Say what you think, keep it technical, and stay patient no matter how bad the code or person. Why: project loyalty beats team popularity; honest reviews are how a team improves. Example:
  ```text
  Reject fearlessly; explain objectively; never make it personal.
  ```

- **Apa.118 Reviewers must not delay or wave through code to protect a release** -> add to §12: If good code is rejected late, that is the author's fault, not the reviewer's; make the fault visible. Why: waving bad code through to protect a date is a false "team player" move that degrades the product. Example:
  ```text
  A delayed release caused by honest rejection is the author's failure, not the reviewer's.
  ```

- **Apa.119 Never estimate a software project's total cost; sell a rate instead** -> add to §12: Replace "how much will it cost?" with "how much software per unit of money and at what quality?". Why: software is never finished; any fixed estimate is a lie that misleads the sponsor. Example:
  ```text
  Metrics for the rate: hits-of-code, bugs, pull requests, test coverage, burn rate.
  ```

- **Apa.120 Trust without control becomes a legalization of chaos** -> add to §12: Pair trust in delegated individuals with explicit control mechanisms (tests, gates, rules). Why: "we trust our programmers to write perfect code" is not a management policy; it is abdication. Example:
  ```text
  Trust = delegate clear tasks fully + define awards/penalties/rules + verify results.
  ```

- **Apa.121 Don't let a manager tell you *how*; require *what* and the rule** -> add to §12: Recognize micromanagement as imperative instructions and counter with a clear goal and explicit rules. Why: declarative management preserves professionalism and matches declarative OO design. Example:
  ```text
  Push back: "Give me the expected result and the success/failure rule; I'll choose the method."
  ```

- **Apa.122 Avoid the seven project sins that make software unmaintainable** -> add to §12: Diagnose maintainability by anti-patterns, untraceable changes, ad-hoc releases, volunteer static analysis, unknown coverage, nonstop dev and undocumented interfaces. Why: maintainability = time for a newcomer to understand; these sins blow it up toward infinity. Example:
  ```text
  Checklist used to audit any repository: 1-7 as above, each must be "clean".
  ```

- **Apa.123 Value simplicity and decomposition over impressive complexity** -> add to §12: An architect's metric is reduced complexity — ≤ 5 rectangles, UML, directed+annotated arrows, no color/creativity. Why: "if I don't understand you, it's your fault"; the job is to decompose, not to display brilliance. Example:
  ```text
  Interface diagram: ≤ 5 rectangles; every line is an arrow AND has a text label.
  ```

- **Apa.124 Treat meetings as a last resort and documents as the deliverable** -> add to §12: Any information exchange should produce a versioned artifact; meetings produce little and cost everyone. Why: only documents/reviews turn paid minutes into a durable product. Example:
  ```text
  Instead of an agenda, circulate a draft .md through PRs until it stabilizes.
  ```

- **Apa.125 Beware fake product-management lessons (the hero myth)** -> add to §12: Prefer risk planning, rules, decisions and consequences over "everyone just loves each other and heroes save the day". Why: real teams have conflicts and incompetence; management is comparison of risks, not cheerleading. Example:
  ```text
  Real PM work: risks, probabilities, impacts, mitigations, rules, punishments, rewards.
  ```

- **Apa.126 Recognize holacracy-style management theater and its cost** -> add to §12: Reject feel-good substitutes (intrinsic rewards, diversity metrics, eco-titles) that mask command-and-control without real rules. Why: it damages people mentally while avoiding explicit, fair accountability. Example:
  ```text
  If there are no explicit awards/penalties/rules, the "inspiration" is just covert control.
  ```

- **Apa.127 Prefer rules and planning over an unpredictable manager** -> add to §12: Define the plan, growth path, risks and exit order up front; decide by rule, not mood. Why: unpredictability is the most demotivating force; a written map makes decisions transparent and respected. Example:
  ```text
  "Make decisions based on rules defined upfront and plans drawn beforehand."
  ```

- **Apa.128 Remove middlemen who add no value to delivery** -> add to §12: Prefer direct connection between money and the people who produce results; strip recruiters/outstaffers where the gap is pure margin. Why: middlemen raise cost and lower motivation without adding artifacts. Example:
  ```text
  "$25 of my $40 should be writing code, not paying an intermediary."
  ```

- **Apa.129 Open source contribution must return value to the contributor** -> add to §12: Before contributing, ask how it improves your resume/reputation, and explicitly request credit in the repo. Why: pure altruism loses motivation; intangible value converts to future pay. Example:
  ```text
  "Add my name and blog link to the contributors list" — inside the pull request itself.
  ```

- **Apa.130 A remote team without results-based pay only amplifies slavery** -> add to §12: Switch to result-based payment before going remote; otherwise remote work adds surveillance, lost knowledge and overtime. Why: motivation must be decoupled from presence first; then remote happens naturally. Example:
  ```text
  Order of operations: results-based pay → remote work, not remote work → surveillance.
  ```

- **Apb.71 "Trust, pay, lose" — audit before you lose the code** -> add to §12: The problem is
  entrusting a project to a team that owns the repository; the solution is a regular independent
  technical review that maintains a Risk List and drives preemptive action. Why: without independent
  inspection, the software quietly becomes theirs, and when they leave you are left with unreadable,
  possibly unavailable code. Example:
  ```text
  before start: hire an independent expert
  every 2 weeks: full review + Risk List + corrective tickets
  ```

- **Apb.72 Can't merge? Give up fast, split, blame wisely, move on** -> add to §12: When a PR stalls,
  close it quickly if the complaints are architectural, resubmit only the accepted subset, file a
  separate bug against master with evidence, and move to easier work; never call for help. Why: forcing
  a merge destroys reputation and time; "a failed PR is a lesson, not a setback." Example:
  ```text
  1) close 2) resubmit the acceptable part 3) bug report on master (no mention of your PR)
  4) move on 5) don't ask them to help you
  ```

- **Apb.73 "CI failures are not related to my changes" is not an excuse** -> add to §12: The problem is
  a red baseline treated as someone else's problem; the solution is to stop, report the broken build,
  and wait, or submit a build fix in its own PR. Why: every diff layered on a broken build raises the
  cost of recovery; no new changes are accepted while the build is red. Example:
  ```text
  red CI -> issue "master broken" -> wait (or a standalone fix PR) -> then branch
  ```

- **Apb.74 "I don't understand the code" is a code-base defect** -> add to §12: The problem is blaming
  yourself for confusing legacy; the solution is to file tickets that demand documentation/source fixes
  and pause until they are addressed. Why: the project (not the person) is responsible for being
  understandable; tickets escalate complexity instead of enshrining it. Example:
  ```text
  "Method X is too complex; I don't know what it does." -> source/documentation ticket
  ```

- **Apb.75 Don't ask them to help you; the school is closed** -> add to §12: The problem is expecting a
  project to teach you its codebase; the solution is to ask the world (Stack Overflow, docs) and turn
  blockers into repository-improving issues. Why: teammates' explanations waste project money, while a
  documentation fix helps everyone forever. Example:
  ```text
  Instead of "Where should I put class X?" -> ticket: "Document the package layout."
  ```

- **Apb.76 Group responsibility is no responsibility** -> add to §12: The problem is solving an
  architect/decision conflict by a democratic vote; the solution is to make each decision a documented
  artifact owned by the architect and have it reviewed by extra pairs of eyes in proportion to risk.
  Why: "quality and responsibility mean nothing unless they are attributed personally"; group blame
  lets everyone escape. Example:
  ```text
  Amend requirements ("each product choice grounded in 4+ alternatives") ->
  architect writes the analysis -> independent reviewer inspects it -> defects fixed.
  ```

- **Apb.77 Stand-ups and daily reports substitute for management via guilt** -> add to §12: The problem
  is a weak management that cannot measure or reward; the solution is an explicit reward/punishment
  system, not guilt rituals. Why: morning stand-ups and CC'd daily reports work only because they
  trigger guilt; that is the wrong instrument for professionals. Example:
  ```text
  Replace: standup + daily email
  With:    "deploy = $120; test fixed = $200; server down 5m = -$500"
  ```

### Additions from real codebases

A cross-repo catalogue of recurring problems and the pattern each repo uses to solve them.

- **`null` cannot be eliminated at the JVM boundary.** Solution: centralize every null decision in a
  decorator (`NoNulls`, `Unchecked`/`IoChecked`/`Checked`) and make the caller choose; throw
  `IllegalArgumentException`/`IllegalStateException` instead of returning null
  (`NoNulls.java:31-45`). Takes uses `Opt<T>`.
- **Statics are sometimes imposed by the platform.** JUnit `@MethodSource` needs a `private static`
  provider, `@SafeVarargs` markers exist, `main` is static. Solution: apply "no public static" to the
  *public API*; tolerate test scaffolding and entry points (Cactoos, rultor `Entry`).
- **Legacy static-API usage cannot be migrated in one commit.** Solution: ratchet with `forbiddenapis`
  `includes` + a PDD puzzle (`pom.xml:404-412`).
- **Coverage thresholds must be a negotiation with reality.** Cactoos 0.61/0.65/0.65 + `CLASS missed
  ≤15`, PIT 75; eo per-module multi-counter gates. Choose achievable-but-binding or the gate gets
  disabled to ship.
- **API stability vs 1.0 cleanup is a real conflict.** Solution: `revapi` allow-list with an
  individually-justified `<ignore>`/`<justification>` per intentional break (Cactoos `pom.xml:150-246`,
  Takes `pom.xml:353-687`).
- **Cross-platform builds break Java/OS assumptions.** Solution: declare capability skips
  (`hone` `skipOnWindows=true`, `skipWithoutDocker=true`), CI matrix of
  ubuntu/windows/macos × one-modern-JDK, and (rultor) a `restrict-windows-build` profile that fails
  fast on Windows with a Qulice-only escape.
- **Documentation and licensing rot.** Solution: bots — `copyrights`, `reuse`, `typos`,
  `markdown-lint`, `yamllint`, `xcop`, `actionlint`, `simian`, `ort`, plus `up.yml` README-version PR.
- **Analyzer contradictions must be resolved explicitly.** Qulice keeps a catalogue of ~20 and
  disables exactly one rule per conflict with a written rationale.
- **Toolchain/JDK drift** (s3auth compiles 1.8 while CI is Java 21 and README says 11;
  `system.properties` says 17). Solution: pin `maven.compiler.release` and align it with CI and docs.
- **License metadata drift** (requs `pom.xml:32` says BSD while every SPDX header says MIT). Solution:
  one license stated identically in POM, SPDX, REUSE, README badge and LICENSE.
- **Hard-coded machine paths in committed POMs** (xembly `prof` embeds
  `/Users/yegor/apps/YourKit_.../libyjpagent.jnilib`, `pom.xml:472`). Solution: keep profiler flags in
  a local profile.
- **Secrets must reach CI without living in git.** Assets repo + copy→commit→push→reset, `trap EXIT`,
  `sensitive:` refusal (rultor/jare/uulu).
- **Test doubles belong in `src/main`, not hidden in test scope** (`FakeBase`, `FakeStatus`, `Mk*`).
- **`argLine` must be empty and composed** so Jacoco/PIT can inject (`<argLine/>` + `@{argLine} …`,
  rehttp `pom.xml:60,206-217`).
- **Random ports, not hardcoded ones, for ITs** (`build-helper:reserve-network-port`; `FtRemote`
  `new ServerSocket(0)`).
- **Flaky tests are a bug.** Solution: nightly repetition harness, `runOrder=random`, retry extension;
  keep fast (default) vs deep (`-Pdeep`) separation.
- **One concern per PR; build-fix PRs are separate.** A red baseline is fixed in its own PR before
  new work.

## 13. Coverage notes

- **The file contains only Volume 1, not Volume 1+2.** Despite the filename
  (`elegant-objects-ocr.md` and its header "vol.1+2"), the OCR is the complete Volume 1
  (Version 1.5, 28 April 2017, ISBN 978-1519166913, "23 practical recommendations", pages 0–233).
  It ends at the Volume 1 index and back cover. **No Volume 2 content is present** — so the
  postulates above cover only Vol. 1's four chapters (Birth, Education, Employment, Retirement).
  Volume 2 topics (e.g. ORM, DTO, serialization, more advanced decoration) must be sourced elsewhere.

- **Sections with thin or no source coverage in this OCR:**
  - §9 Build, CI/CD and releases — not covered (no build/CI/release material in Volume 1).
  - §10 Project and repository management — not covered (no ticketing, branching, release-script,
    semver or GitHub material in Volume 1).
  - §8 Static analysis — the book gives *metrics and style rules* (250 LOC, <5 public methods, <4
    fields, no comments, declarative style) but names **no static-analysis tooling** (no Qulice,
    Checkstyle, PMD, FindBugs, Jacoco). Those must come from the project's other sources.
  - §11 Tools and libraries — only incidental mentions: Lombok `@EqualsAndHashCode`, JUnit/Hamcrest
    `assertThat`, Mockito (as an anti-pattern), jcabi-aspects `@RetryOnFailure`, Apache Commons
    `FileUtils` (as something to wrap), `java.util.Optional` (rejected), `AutoCloseable`. No Maven,
    Takes, Cactoos, Rultor or Docker material appears.

- **OCR quality problems (pages are present but text/code is often mangled):**
  - Mangled tokens throughout: `f{` for `{`, `Q`/`()`/`Q)` for `()`, `dir`/`dlr` for `dollars`,
    `nun`/`num` for `num`, `mul`/`mu1`, `1`/`l`/`I` confusion, `RAT`/`RATI`/`RATI` for RAII,
    `RAT)` for `RAII`. These were normalized in the examples above.
  - Heavily garbled code blocks (braces/colons lost, statements merged) on roughly pages 36–40,
    51–53, 55–63, 67–74, 100–113, 118–146, 154–165, 175–181, 183–185, 200–219, 221–223.
  - Essentially blank/near-blank pages: 1, 5, 9, 11, 17, 187, 225, 231, 232 (page-number-only or
    "IXG"/"lO" artifacts).
  - The table of contents (pages 6–8) OCRs section numbers badly (e.g. "2.9.1/2.9.2" appear under
    2.5, "2.9 Keep interfaces short" appears where 2.9 is meant), but the body text disambiguates:
    2.5 = "Don't use public constants" (2.5.1 coupling, 2.5.2 cohesion) and 2.9 = "Keep interfaces
    short; use smarts".
  - Nothing in the book failed to fit the required sections; the only mismatch is that several
    required sections (§9, §10, §8 tooling, §11 build libs) are simply absent from Volume 1 and are
    flagged as "Not covered by this source" rather than invented.

## Cross-source notes

### Contradictions and tensions between sources

- **Method overloading.** The book prescribes overloading (`B2.6`, `B2.13`, `B4.6`) and calls it a
  fundamental OOP feature; article `Acs.12` bans overloading outright and demands a distinct name or a
  decorator for every behavior.
- **Method chaining / Law of Demeter.** `Acs.18` argues chaining such as `book.pages().last().text()`
  is legitimate OO (the LoD only bans getters); `Acob.29` and `Acob.22` say fluent chains must be
  replaced by bottom-up decorator nesting, and the book (`B5.9`, `B8.6`) demands declarative
  composition. Same code, opposite verdicts.
- **Prefixes vs compound names.** `Acs.11` and `Acob.19` *recommend* entity-prefixed class names
  (`RqFake`, `RsWithStatus`, the Takes 10 prefixes), while `Acs.10` bans compound names in favour of a
  single noun and the book (`B4.2`) dislikes suffixes. The prefix deliberately repeats a shared entity
  noun across many classes, which some reviewers read as a compound-name violation.
- **"Let it crash" vs "no null".** `Acs.21`, `Acob.34`, `Atq.10` say core methods must not defend
  inline and should crash on bad input; the book (`B6.1`-`B6.3`) forbids null on both sides. They are
  compatible only if "crash" means *fail fast on non-null invalid state*, not *tolerate null*.
  `Acoa.29` even permits `if (x == null)` guards for JDK/third-party APIs, whereas the book (`B6.2`)
  prefers to ignore the null and let the NPE happen.
- **AOP / annotations.** The book embraces AOP for cross-cutting concerns (`B6.11`, `B11.4`,
  `@RetryOnFailure`) and `Atq.7`/`Atq.16`/`Atq.71` follow it; `Adt.2` calls behavior-carrying
  annotations "a big mistake" and `Adt.21` insists retry be an explicit visible decorator, not an
  annotation. `Adt.65` allows weaving only as a reluctant fallback.
- **Test-first vs bug-driven.** `Ada.25` says start from a test before the implementation; `Atq.41`
  says implement, deploy, let users break it, then write the test (and `Atq.17`/`Atq.18` frame tests as
  destructive bug hunting). The book's `B7.12` sits with the bug-driven camp.
- **Test-class size.** The book (`B1.21`, `B8.10`) applies the 250-LOC / five-method limits to test
  code; `Atq.37` explicitly says a 5000-line `FooTest` is fine because it is a container of scripts.
- **Documentation.** The book (`B4.14`, `B8.4`) and `Apa.31` allow documenting *external* interfaces;
  `Acs.27` prohibits all documentation comments, including Javadoc, and proposes an automatic
  interpretability gate instead.
- **Estimating cost.** `Apa.70`/`Apa.119` refuse total-cost estimates and demand a rate; `Apa.49`,
  `Apb.31`, `Apb.33` simultaneously fix a price for every micro-task. A rate can be fixed while a
  project total cannot, but the two rules pull in opposite directions in front of a client.
- **Dependency versions.** `Ada.31`/`Ada.44` prefer dynamic version ranges for trusted authors, while
  `Apb.23`/`Adt.32` want maximal strictness and reproducibility; a time-bomb risk trades against a
  conflict risk.
- **Managed repo vs raw S3.** `Adt.46` deploys to a hand-rolled S3 Maven repo; `Apb.67` argues a
  managed cloud repository should replace it.

### Topics covered by only one side

- **Only the book (not in the articles):** the core postulates `B1.1`-`B1.25` (class as object
  factory, representative objects, ≤4 encapsulated objects, no utility classes/singletons, no
  getters/setters, no reflection, OOP-vs-FP); the full constructor discipline `B2.x`; the exception
  philosophy `B6.x` (checked-only, one type, chain-and-rethrow, recover once, Null Object, rejecting
  `Optional`); `B5.2`/`B5.12` abstract-class refinement; `B11.7` C++ RAII example; `B11.1` Lombok
  `@EqualsAndHashCode`; and the book's ground-truth coverage/OCR notes (§13).
- **Only the articles (explicitly absent from the book):** everything in §9 build/CI/CD (Maven
  wrapper, Sonatype OSRH signing, Docker-per-build, Rultor/AppVeyor, `build-helper` port reservation,
  failsafe phases, Liquibase, PaaS pushes); §10 project/repository management (ticket-first, PDD
  puzzles, Zerocracy-style roles/rewards, micro-tasking, independent reviews, README canon, commit
  traceability); §8 tooling names (Qulice, jPeek, Jacoco coverage + mutation gates, `hoc`, `tdx`,
  `0pdd`, Requs); most of §11 (Cactoos, Takes, jcabi-* family, Xembly, Saxon/XSLT 2.0, DI-container
  rejection); and testing tooling (JUnit 5 tags/extensions, fake objects vs mocks, `Threads` latch
  tests, `MkGrizzlyContainer`, `XhtmlMatchers`).

## 13. Reference architecture of an EO Java project

Synthesised from the seven code analyses (`knowledge/elegant-objects-java.md`, `knowledge/elegant-objects-java.md`, `knowledge/elegant-objects-java.md`,
`knowledge/elegant-objects-java.md`, `knowledge/elegant-objects-java.md`, `knowledge/elegant-objects-java.md`, `knowledge/elegant-objects-java.md`). This is the
shape a *real* yegor256-style EO Java repository takes; the book itself says nothing about Maven or
directory layout.

### 13.1 Repository root (what lives at the top level)

Minimal and stable. A typical root contains only:

```
pom.xml                 # the only build entry point
README.md               # the external contract (badges, usage, contribute)
LICENSE.txt             # one license, matching SPDX headers
LICENSES/<id>.txt       # REUSE licence texts
REUSE.toml              # non-source file → licence mapping
.rultor.yml             # merge + release contract
.pdd                    # Puzzle-Driven Development rules
.0pdd.yml               # puzzle → GitHub issue bot
renovate.json           # dependency updates (NEVER Dependabot)
.gitattributes          # * text=auto eol=lf ; *.java ident ; *.xml ident
.gitignore              # target/, IDE, node_modules/, .claude/
.github/workflows/*     # many small single-purpose workflows
.mvn/                   # wrapper + jvm.config  (only in newer repos: eo)
src/                    # code + tests + ITs + site
```

Observed: **no `Dockerfile`** in most repos (Rultor pins a Docker image), **no `CONTRIBUTING.md`** in
Cactoos/Takes/Qulice/eo/s3auth/xembly (guidance is a README section), **no `.github/ISSUE_TEMPLATE` or
PR template**, **no Makefile/Justfile** (Maven + Rultor only). rultor additionally ships
`rultor_schema.json`; takes/requs/eo ship `CITATION.cff`; xembly/takes ship a website under
`src/www.*`/`src/site`.

### 13.2 Source tree layout

```
src/
  main/java/<groupId path>/      # production .java, package = domain, never "layer"
  main/resources/                # classpath assets; config templates, XSL
  test/java/<same package>/     # *Test.java, 1:1 with main packages
  test/resources/               # fixtures, forbidden-apis.txt, externalised expectations
  it/<name>/                    # standalone Maven "example app" driven by maven-invoker-plugin
  site/                         # Maven site sources
  www.<host>/                   # static website deployed to gh-pages
  main/xsl, main/requs, ...     # generated/domain resources wired into the build
```

Rules observed:

- **Package = domain concept, not layer.** Cactoos packages are `bytes, collection, exception, func,
  io, iterable, iterator, list, map, number, proc, scalar, set, text, time`; Takes uses `tk`, `rq`,
  `rs`, `facets`, `http`, `misc`, `servlet`. There is no `controllers/services/repositories` package
  anywhere and **no `-er` package**.
- **One `package-info.java` per package** (main and test), carrying a one-line Javadoc and any PDD
  `@todo` puzzles (Cactoos has 17, 9 with puzzles). `pom.xml`/Qulice requires it.
- **Tests mirror main 1:1** by package and by class name (`AndTest` for `And`). `FooTest` tests
  `Foo`; `FooITCase` tests integration; anything else goes to `support/` or `it/` packages with no
  main counterpart.
- **Test doubles live in `src/main` when they are legitimate objects** (rehttp `FakeBase`/`FakeStatus`;
  s3auth `Mk*`, `*Mocker`; jare `io.jare.fake.Fk*`) — they are part of the product contract, not
  test-only hacks.
- **Integration tests get their own tree** (`src/it/<project>`), each with its own `pom.xml`,
  `LICENSE.txt`, `invoker.properties` (`invoker.goals = clean verify`) and a `verify.groovy` that
  asserts on build output (Takes `file-manager`, Qulice's 22 ITs, requs plugin ITs, xembly
  `src/it/saxon` + `src/it/xerces`).

### 13.3 Single-module vs multi-module

- **Single module** (Cactoos, Takes, Qulice plugin, xembly, jare, rehttp): one `pom.xml`,
  `packaging=jar`; version always `1.0-SNAPSHOT`/`2.0-SNAPSHOT`; the release bot injects the tag.
- **Multi-module / aggregator** (eo 8 modules, s3auth 3, requs 3): root `packaging=pom` with
  `<modules>`; dependency-ordered layers (parser → transforms → plugin → runtime → integration-tests;
  or hosts → relay → rest).
- **Single-module for a web app** (Takes, rultor, jare, s3auth, rehttp) plus a nested invoker IT.
- Generalizable: decompose aggressively — the largest acceptable code base is ~50,000 lines
  (`Ada.34`); small repos permit maximum lint strictness, fast builds and cheap CI.

### 13.4 `pom.xml` structure

```xml
<project>
  <parent>                          <!-- 1. inherit the quality parent -->
    <groupId>com.jcabi</groupId>
    <artifactId>parent</artifactId>
    <version>0.73.4</version>
  </parent>
  <groupId>org.example</groupId>
  <artifactId>example</artifactId>
  <version>1.0-SNAPSHOT</version>
  <packaging>jar</packaging>
  <name>...</name>
  <description>...</description>
  <licenses>MIT</licenses>

  <properties>
    <maven.compiler.release>17</maven.compiler.release>  <!-- or jdk.version 1.8 -->
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
  </properties>

  <dependencies>                    <!-- 2. runtime deps minimal; test/provided the rest -->
    ...
  </dependencies>

  <build>
    <plugins>                       <!-- 3. compiler, surefire, failsafe, invoker, license, revapi -->
      ...
    </plugins>
  </build>

  <profiles>
    <profile><id>qulice</id>        <!-- 4. static analysis is a profile -->
      <build><plugins><plugin>
        <groupId>com.qulice</groupId>
        <artifactId>qulice-maven-plugin</artifactId>
      </plugin></plugins></build>
    </profile>
    <profile><id>jacoco</id>...</profile>
    <profile><id>deep</id>...</profile>
    <profile><id>sonar</id>...</profile>
    <profile><id>sonatype</id>...</profile>
  </profiles>
</project>
```

Observed conventions:

- **The parent owns the toolchain.** Qulice, Jacoco, compiler, enforcer and release plumbing come
  from `com.jcabi:parent`; child POMs override only what they must. Versions for the app's own deps
  are pinned explicitly, or centralized in `<dependencyManagement>`/a BOM (`s3auth` AWS SDK BOM
  2.44.4, eo `junit-bom`, `dependencyManagement` at `pom.xml:94-309`).
- **Scopes tell the story.** A library declares `provided`/`test` only (xembly: Lombok + xml-apis
  provided; everything else test). A web app keeps exactly one hard compile dep (`cactoos`) and marks
  optional integrations `optional`.
- **`maven-compiler-plugin` `release` must match CI and docs** (the s3auth 1.8-vs-21-vs-11 drift is
  the counter-example). eo pins the compiler to 3.8.1 and excludes it from Renovate.
- **Named profiles, invoked by id:** `-Pqulice` (static analysis + forbiddenapis), `-Pjacoco`
  (coverage), `-Psonar`, `-Pdeep`/`-Pperformance` (slow tests), `-Psonatype` (Central deploy),
  `-Psite`, app-specific (`-Ptakes`, `-Pjare`, `-Ps3auth`, `-Prultor`, `-Pheroku`, `-Pdynamodb`).
- **Profiles gate *extra* quality only.** `mvn install` does not run Qulice; CI/merge/release always
  pass `-Pqulice` — "opt-in locally, mandatory in CI".
- **Verification past tests:** `revapi` API-compat, `pitest` mutation, `jacoco` gates,
  `maven-verifier` invariants, `license-maven-plugin:check-file-header` at `verify`,
  `maven-invoker-plugin` for IT example apps.

### 13.5 Naming conventions in a real tree

- Interface = noun (`Request`, `Response`, `Scalar`, `Text`, `Host`, `Bucket`, `Domain`, `Base`,
  `Status`, `Rule`, `Facet`, `Take`).
- Implementation = short entity prefix + specifier: `Rq*`/`Rs*`/`Tk*`/`Fk*` (Takes), `Default*`
  (s3auth/requs), `Mk*` (mocks), `Dy*` (Dynamo), `Cd*` (cached), `Tr*`/`Xe*` (xembly).
- Decorator = `XxxWithYyy` / `XxxEnvelope` / `Smart*` / `Fast*` / `Synced`.
- No `Manager`, `Helper`, `Util`, `Controller`, `Handler`; the accepted `-er` nouns are domain
  entities like HTTP `Header` (and legacy `Compiler` in requs, to be migrated).
- Longest observed class name stays short (`RqWithDefaultHeader`); a rich set of small classes is a
  virtue, not a defect.

## 14. Reference CI/CD pipeline

### 14.1 The two-pipeline model

- **GitHub Actions = the PR gate** (lint + build + tests + coverage upload). It never tags or deploys.
- **Rultor chat-ops = the only merger/deployer** (`.rultor.yml`). Read-only `master`; PR-only merges;
  every merge is GPG-signed by the bot.

### 14.2 Canonical `mvn.yml` (the build gate)

```yaml
name: mvn
on:
  push: { branches: [master] }
  pull_request: { branches: [master] }
permissions: { contents: read }
concurrency: { group: mvn-${{ github.ref }}, cancel-in-progress: true }
jobs:
  build:
    strategy:
      matrix:
        os: [ubuntu-24.04, windows-2022, macos-15]
        java: [23]
    runs-on: ${{ matrix.os }}
    timeout-minutes: 15
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-java@v4
        with: { distribution: temurin, java-version: ${{ matrix.java }} }
      - uses: actions/cache@v4
        with:
          path: ~/.m2/repository
          key: ${{ runner.os }}-jdk-${{ matrix.java }}-maven-${{ hashFiles('**/pom.xml') }}
          restore-keys: ${{ runner.os }}-jdk-${{ matrix.java }}-maven-
      - run: mvn --errors --batch-mode clean install -Pqulice
```

Observed variants: Cactoos/Takes run `-Pqulice` (Takes adds `-Pdeep`); eo runs 4 OSes × Java 26;
Qulice runs `mvn install` then `mvn verify -Pqulice -DskipTests -DskipITs`; xembly adds
`-Dhone.skip=true`.

### 14.3 Cross-cutting CI rules (from `knowledge/elegant-objects-java.md`)

- `permissions: contents: read` by default; grant `write`/`id-token` only where needed.
- `concurrency: {group: <wf>-<ref>, cancel-in-progress: true}` on nearly every workflow.
- Maven cache key `${runner.os}-jdk-${java}-maven-${hashFiles('**/pom.xml')}` with `restore-keys`;
  **restore-only caches on PR workflows** so PRs never evict master's cache.
- `timeout-minutes` on every job (bounded, not the 360 default).
- `fetch-depth: 2` where only merge-base is needed (documented in a comment).
- Before install, `rm -rf ~/.m2/repository/org/<group>` to force re-resolution of sibling snapshots.
- Heavy jobs go to ephemeral self-hosted runners (EC2) that always tear down (`if: always()`).

### 14.4 The lint/policy workflow fleet

Each concern is its own small workflow (never one mega-job): `codecov` (master-only, `fail_ci_if_error:
true`), `pdd`, `reuse` (+`copyrights`), `typos`, `xcop`, `yamllint`, `actionlint`, `markdown-lint`,
`shellcheck`/`bashate`, `simian` (duplication, `-threshold=`), `ort` (dependency-license audit,
`fail-on: violations`), `sonar`/`codenarc`/`infer`/`scancode` (language-dependent), `ladder`/`counts`
(regression/metrics), `up` (README-version sync), `zerocracy`/`telegram` (reporting). Every YAML
carries an SPDX header and `# yamllint disable rule:line-length`.

### 14.5 Rultor skeleton

```yaml
docker:
  image: yegor256/java
readers:
  - "urn:github:526301"
assets:
  settings.xml: yegor256/home#assets/<repo>/settings.xml
  pubring.gpg: yegor256/home#assets/pubring.gpg
  secring.gpg: yegor256/home#assets/secring.gpg
install: |-
  pdd --file=/dev/null
merge:
  script: |-
    mvn clean install -ntp -Pqulice --errors --settings ../settings.xml
    mvn clean site -Psite --settings ../settings.xml
release:
  pre: false
  sensitive:
    - settings.xml
  script: |-
    [[ "${tag}" =~ ^[0-9]+\.[0-9]+\.[0-9]+$ ]] || exit -1
    mvn -ntp versions:set "-DnewVersion=${tag}"
    git commit -am "${tag}"
    mvn clean deploy -Psonatype -Pqulice --errors --settings ../settings.xml
```

Release procedure: (1) semver-tag validation; (2) `versions:set` + commit the tag; (3) build
`clean deploy -Psonatype` with GPG-signed artifacts; (4) app repos additionally stamp the git short
hash into `META-INF/MANIFEST.MF`, `git push -f` to Dokku/Heroku, then `curl --retry` the live URL;
(5) `site-deploy` is best-effort; (6) if any step fails the log is reported back into the ticket.

### 14.6 Quality gates (must all fail the build)

- **Build/tests:** unit (`*Test`, surefire) then integration (`*ITCase`, failsafe); deep tests via a
  profile.
- **Static analysis:** Qulice (`mvn verify -Pqulice -DskipTests -DskipITs`) — Checkstyle + PMD +
  ErrorProne + custom EO rules.
- **Coverage:** Jacoco thresholds in the POM, uploaded to Codecov with `fail_ci_if_error: true`.
- **Mutation:** PIT threshold 75–80.
- **API compatibility:** Revapi with justified exceptions.
- **Hygiene:** SPDX/REUSE, license header, typos, XML/YAML/Markdown lint, duplication, dependency
  audit, and (recently) a VCS-hash/version manifest check.
- **Process:** PDD puzzles tracked; `up.yml` keeps docs versions honest.

### 14.7 Runtime/deploy declaration

App repos keep their runtime in-repo: a `Procfile` (Heroku) and/or `deploy.sh` (a manual/DR fallback
that commits the secret, pushes, and `trap`s the secret away on any exit), `app.json` health checks,
and `nginx.conf.sigil` for Dokku. The release ends only when the live endpoint answers.

## 15. Tool & dependency catalogue

"what / why / when to use / when not" — distilled from the seven code analyses. Versions are those
observed and are illustrative, not prescriptive.

### Build & toolchain

| Tool | What / why | When to use | When not |
|---|---|---|---|
| **Maven** (+ `com.jcabi:parent`) | Declarative build; parent centralizes plugins/quality. | Always; keep child POM thin. | Do not re-declare what the parent enforces. |
| **Maven wrapper / `.mvn`** | Reproducible Maven version. | Newer repos (eo commits `.mvn`). | Older repos ship none; CI installs Maven. |
| **`maven-compiler-plugin` (`release`)** | Pin bytecode target. | Every repo; match CI + docs. | Don't leave it implicit/drifted (s3auth 1.8 vs 21). |
| **`dependencyManagement` / BOM** | One place for versions (poor-man's BOM). | Multi-module or many deps. | Don't pin the same literal in every module. |
| **`maven-invoker-plugin`** | Run standalone example/IT Maven projects. | Plugin/CLI-style projects, cross-impl ITs. | Simple libraries without external usage samples. |
| **`maven-assembly-plugin`** | Fat/runnable JAR. | Executables (`requs-exec`). | Libraries (consumers pick deps). |
| **`flatten-maven-plugin`** | Flatten POM before deploy. | Multi-module Maven-Central releases. | Minimal single-module libs. |
| **`build-helper:reserve-network-port`** | Random free port for ITs. | Any server IT (DynamoDB, Tomcat, Takes). | Never hardcode ports. |
| **`download-maven-plugin` / `exec-maven-plugin`** | Fetch + run CLI-only tools (Nutch, DynamoDBLocal). | Wrapping binary distributions. | When a pure-Java lib exists. |
| **`sass-maven-plugin`** | Compile SCSS→CSS at `generate-resources`. | Web apps with stylesheets. | Back-end libs. |
| **`license-maven-plugin`** | Enforce license file header at `verify`. | Every repo. | — |
| **`versions-maven-plugin`** | `versions:set` from the release tag. | All tagged releases. | Never edit POMs by hand. |
| **`nexus-staging-maven-plugin` / `maven-gpg-plugin` / source+javadoc** | Signed Maven-Central release. | Public libraries. | Internal apps (Dokku/Heroku deploy instead). |
| **`maven-s3-wagon`** | Private Maven repo on S3. | Legacy private artifact hosting. | Prefer a managed cloud repo (`Apb.67`). |
| **`hone-maven-plugin`** | EOLANG optimizer. | objectionary/eo and Cactoos experimental. | Ordinary EO Java; skipped on Windows/without Docker. |

### Static analysis, coverage, compatibility

| Tool | What / why | When to use | When not |
|---|---|---|---|
| **Qulice** (Checkstyle+PMD+ErrorProne+custom) | The enforceable EO quality wall; one `verify` goal. | Every Java repo, mandatory in CI. | Never as an advisory report; must fail the build. |
| **`forbiddenapis`** | Ban static APIs in already-migrated packages. | Ratcheting legacy code. | Don't ban globally on day one. |
| **Jacoco** | Coverage measurement + POM gates. | Always; multi-counter thresholds. | Don't set 100%; choose achievable-but-binding. |
| **PIT (pitest)** | Mutation testing (proves tests detect change). | Libraries and critical cores (75–80%). | Trivial glue code; very slow suites. |
| **Revapi** | Binary/source API compatibility gate. | Public libraries with semver. | Apps with no external API. |
| **SpotBugs** | Bug-pattern analysis. | Libraries (xembly); Qulice already includes FindBugs-style rules. | Redundant alongside Qulice unless extra categories needed. |
| **ArchUnit** | Architecture invariants (packages, hierarchy). | Large/multi-module codebases (eo). | Tiny single-module libs. |
| **jtcop** | Lints *test* code (naming/assertions). | Mature suites (eo). | Where test style differs deliberately. |
| **SonarCloud / Codacy / Infer / scancode** | Dashboards, extra bug/security/license scanning. | Larger projects wanting a quality gate/badges. | As a replacement for a failing build. |
| **jPeek** | Cohesion metrics to validate EO claims. | Research/refactoring audits. | As a runtime gate. |

### Testing

| Tool | What / why | When to use | When not |
|---|---|---|---|
| **JUnit 5 (Jupiter)** | Test platform; tags, extensions, params. | Always. | — |
| **Hamcrest `assertThat`** | Declarative, composable assertions. | Always, with a reason string. | Raw `Assert.*` (Qulice forbids). |
| **cactoos-matchers / jcabi-matchers / XhtmlMatchers** | OO matchers (`HasValue`, `IsText`, XPath). | OO assertions, XML/HTTP. | — |
| **Fakes (`Fake*`, `Mk*`, `*Mocker`, `Fk*`)** | Reusable real implementations of interfaces. | Preferred over mocking, shipped in `src/main`. | Don't hide genuinely internal classes. |
| **Mockito** | Interaction mocking. | Rare spy cases only (Cactoos 1 hit). | As default; prefer fakes. |
| **jqwik** | Property-based testing. | Escaping/serialization/parsers. | Example-only unit tests. |
| **junit-pioneer / `@TempDir` / `ParameterResolver`** | Inject env/temp/prereqs. | Integration-ish unit tests. | Trivial tests. |
| **rerunner-jupiter / repetition scripts** | Flaky-test retry + nightly repetition. | Known-flaky ITs; nightly concurrency. | As a mask for real bugs. |
| **JMH** | Micro-benchmarks. | Performance-critical code. | Regular CI gates (noisy). |
| **JCabi HTTP (`MkContainer`, `MkGrizzlyContainer`) / `FtRemote`** | Real HTTP stack in tests. | HTTP clients/servers. | Pure logic tests. |
| **DynamoDBLocal / H2 / reserved ports** | Real dependency for ITs. | Persistence ITs. | Unit tests. |

### Runtime libraries (OO stack)

| Library | What / why | When to use | When not |
|---|---|---|---|
| **Cactoos** | OO primitives replacing Guava/Commons (`ListOf`, `BytesOf`, `Sticky`/`SyncScalar`). | Default primitive/IO library. | Don't wrap it back into getters; don't add Guava alongside. |
| **Takes** | OO web framework (`Take`/`Request`/`Response`). | Web apps, REST, chat-ops. | Heavy Spring-style MVC shops (migration risk). |
| **jcabi-aspects** | AOP decorators (`@Immutable`, `@Cacheable`, `@Loggable`, `@Timeable`, `@RetryOnFailure`). | Cross-cutting concerns where explicit decorators are impractical. | Where a visible decorator suffices (book prefers it). |
| **jcabi-http / jcabi-xml / jcabi-jdbc / jcabi-dynamo / jcabi-s3 / jcabi-ssh / jcabi-manifests / jcabi-log** | OO wrappers over HTTP, XML, SQL, Dynamo, S3, SSH, manifests, logging. | Persistence/integration seams. | If only used once, wrap the JDK call yourself. |
| **Xembly** | Imperative XML generation (`Iterable<Directive>`). | Build XML without JAXB/getters. | Parsing (use jcabi-xml). |
| **Saxon-HE** | XSLT/XPath 2.0+ (JDK is 1.0). | Transformations needing 2.0. | Plain 1.0 stylesheets. |
| **AWS SDK v2 / v1** | S3/CloudWatch/DynamoDB. | Apps using AWS; hide behind interfaces (`Dy*`). | Domain code — never let AWS types leak. |
| **Lombok** | `@EqualsAndHashCode`, `@ToString`, `@Immutable` markers. | Immutable value objects to cut boilerplate. | As a substitute for behavior; keep it `provided`. |
| **Guava / Apache Commons** | Legacy utilities. | Only inside a dedicated wrapper class. | New EO code (use Cactoos/jcabi). |
| **Validation API (`javax.validation`)** | Bean-validation annotations. | Legacy apps. | New EO code (use validating decorators). |
| **Guice / Spring / Dagger** | DI containers. | — | EO (constructor composition + `new` instead). |
| **ORM (Hibernate, etc.)** | Object-relational mapping. | — | EO ("SQL-speaking objects"). |

### Process / repo / CI tooling

| Tool | What / why | When to use |
|---|---|---|
| **Rultor** | Chat-ops merge + release bot on a pinned Docker image. | The only deployer; read-only master. |
| **PDD + 0pdd** | `@todo` puzzles → GitHub issues. | Every repo; `install: pdd`. |
| **Renovate** | Dependency updates with rules. | Every repo (never Dependabot). |
| **REUSE + SPDX + `copyrights`** | Machine-checkable licensing. | Every file, every repo. |
| **Codecov / SonarCloud** | Coverage/quality visibility. | Larger projects; gate with `fail_ci_if_error`. |
| **Qulice ITs (`src/it` + `verify.groovy`)** | Assert the tool itself fails bad code. | Tool/plugin projects. |
| **hoc / tdx / 0pdd badges** | Hits-of-Code, test dynamics, puzzle count. | Dashboards/README badges. |
| **Requs / Markdown docs** | Machine-checkable SRS / versioned docs. | Requirements-heavy projects. |
| **AI agents (PDD chores)** | Bug rewording, small refactor PRs, docs sync. | One small concern per PR (`Apb.70`). |

## 16. Pragmatic deviations

Real EO repos are **EO-aspirational but not puritan**. This section records where the code deviates
from the strict canon, why, and the recommended default. Encoded rule: *state the default and the
explicit, narrow exception; never silently break the rule.*

| Area | Strict canon (book) | Observed deviation (evidence) | Why | Recommended default for the skill |
|---|---|---|---|---|
| **Static methods** | No statics, not even private (`B1.9`). | `Entry.main` public static, `pulse()` private static, static `Pattern` (`rultor Entry.java:78,179`, `SafeUser.java:22-24`); xembly `Verbs` static parser with static maps (`Verbs.java:30-56`); requs `Compiler.decor`, `main`; `@MethodSource`/`@SafeVarargs` in tests. | Entry points, compile-time constants, test scaffolding, legacy code. | **[MUST]** no `public static` behaviour in the public API; **[SHOULD]** no static fields except `private static final` constants; tolerate `main`, annotation-required statics, test providers. |
| **Utility classes** | Never (`B1.10`). | `Entrance`/`Main` are classes of one static method with a private ctor and a `// utility class` comment (requs `Entrance.java:26-33`). | Launchers/CLI entry points. | **[MUST]** not in domain code; **[NICE]** a launcher may be a thin static `main` delegating to objects. |
| **Lombok** | Prefer explicit `equals`/`hashCode` from state (`B11.1`). | Heavy `@EqualsAndHashCode`/`@ToString`/`@ToString(of=...)` (Takes, xembly, rultor, s3auth, requs). | Cuts boilerplate on immutable values. | **[SHOULD]** annotate value objects with `@EqualsAndHashCode`/`@ToString`, keep Lombok `provided`; don't use it to inject behavior. |
| **Javadoc/comments** | No comments for internals; names carry meaning (`B8.4`); `Acs.27` bans all docs. | Qulice **requires** full Javadoc on every type/method/field with `@param`/`@return`/`@since` (Takes 424 `@since`; s3auth 507 `/**`; xembly 150). | This is the Qulice reality and the contradicting article `Acs.27`. | **[MUST]** document external interfaces and satisfy Qulice; **[SHOULD]** no inline narration of internals. |
| **`null`** | Never (`B6.1`–`B6.3`). | Internal nulls: Takes ~62 (`HttpException.java:39`, `TkFallback.java:213-214`); `MkHost.stats()` returns null; `DefaultHost` null-checks; requs `Retry.call` may return null. | Lazy caches, legacy JDK signatures, mocks. | **[MUST]** no null crosses the public contract; centralize in decorators (`NoNulls`); **[NICE]** JDK-boundary null checks allowed. |
| **`-er` names** | Ban them (`B1.12`). | HTTP `Header` (Takes, an entity noun); legacy `Compiler`, `OptionParser`, `DirectoryListing`, `Locator`, `Verbs`. | Some nouns are genuine entities; legacy. | **[MUST]** no new `-er` job titles; accept fossilized entity nouns; treat legacy as migration targets. |
| **Class finality** | `final` or `abstract` only (`B1.4`). | `RsWrap`/`TkWrap` are concrete non-final classes with `final` delegate methods (`RsWrap.java:28`, `TkWrap.java:68`). | Framework extension points (envelope bases). | **[MUST]** `final` (or `abstract`) everywhere except a documented envelope base whose methods are `final`. |
| **Mutable objects** | Never (`B3.15`). | xembly `Directives` is a documented mutable, thread-safe builder; Maven Mojo public fields (`CompileMojo.java:41`); Takes `Opt`, lazily cached internals. | Fluent builders, Maven plugin API, lazy caches. | **[MUST]** domain objects immutable; **[SHOULD]** confine mutable/builder types to Mojo APIs and internal caches. |
| **Getters** | None; tell-don't-ask (`B4.11`). | Model interfaces expose `Domain.owner()/name()`, `Host.syslog()`, `stats()` (s3auth/requs). Bean-style `getX` mostly absent. | Persistence/model seams. | **[MUST]** no `get`/`set` prefixes; **[SHOULD]** behavior over field access even on models. |
| **Casting / `instanceof`** | None (`B1.14`). | `equals` implementations use `instanceof`+cast (xembly `DefaultHost.equals`, `Directives.copyOf`); rultor `Credentials.Simple.class.cast` (`Entry.java:167-170`); xembly `Attr.class.cast`. | JDK `equals(Object)` contract, legacy. | **[MUST]** none in new code except the unavoidable `equals`/`hashCode` type test; **[SHOULD]** prefer `Digitizable`-style contracts. |
| **Reflection / annotations carrying behavior** | Forbidden (`B1.14`, `Adt.2`). | jcabi-aspects `@Cacheable`/`@Loggable`/`@RetryOnFailure`; validation-api `@NotNull`. | Cross-cutting concerns in legacy services. | **[NICE]** only as a reluctant fallback; prefer visible decorators; **[MUST]** no `Class.forName` in business logic. |
| **Test single-statement / no fixtures** | One `assertThat`, no `@Before` (`B7.3`, `Atq.21`). | Real tests use `@BeforeEach` + `Assumptions` (rehttp), loops, `Assertions.assertThrows` (21× xembly, 8× s3auth), `assertThrows` OO matchers. | Data-driven/boundary tests. | **[MUST]** one semantic assertion per test and no shared mutable fixtures; **[NICE]** parameterized/assumption patterns are fine; don't use Mockito by default. |
| **`@Tag` fast/deep** | Not in the book; article (`Atq.30`). | Takes 42 `@Tag("deep")`; eo tags `slow`/`snippets`; requs/rehttp have **0** tags. | Older repos split by naming/profile instead. | **[SHOULD]** tag slow tests and exclude by default; **[NICE]** not mandatory in small/legacy repos. |
| **Method overloading** | Encouraged (`B2.6`, `B2.13`, `B4.6`). | Article `Acs.12` bans it outright. | Sources disagree. | **[SHOULD]** prefer a distinct noun-named method; overload only genuine convenience that funnels to one primary (book wins by default). |
| **Builder pattern** | Avoided (`B5.8`). | xembly `Directives` builder; `with()` chains in xembly/Cactoos. | Fluent APIs. | **[MUST]** no giant builders; **[NICE]** immutable `with(...)` chains that return new objects are acceptable. |
| **Dependency versions** | Trust-based dynamic vs fixed (`Ada.31`). | All repos pin exact versions; eo/Renovate adds `ignoreDeps`/`allowedVersions` guard-rails. | Reproducibility vs conflict risk. | **[SHOULD]** pin exact versions + Renovate guard-rails by default; dynamic ranges only for trusted, semver-disciplined authors. |
| **Toolchain/JDK alignment** | Implied by maintainability. | s3auth compiles 1.8, CI 21, README 11, `system.properties` 17; rehttp runtime 1.8 vs CI 21; Takes targets 1.8 but CI 23/25. | Legacy support windows. | **[MUST]** `maven.compiler.release`, CI JDK and documented JDK must be stated and aligned; if they differ, document why. |
| **License metadata** | One license (`code-requs` lesson). | requs POM says BSD while SPDX/REUSE say MIT. | Historical. | **[MUST]** POM, SPDX, REUSE, README badge and LICENSE all agree. |
| **Static-analysis suppressions** | None needed if code is clean. | `@checkstyle ... (500 lines)`, `@SuppressWarnings("PMD.ProhibitPublicStaticMethods")`, revapi justified ignores, `forbiddenapis` `includes` + `@todo`. | Legacy/boundary code. | **[MUST]** every suppression/exclusion is narrow, carries a reason and (ideally) a ticket; never blanket-suppress, never disable a whole rule silently. |
| **README as marketing** | Document external interfaces (`B8.4`, `Apa.31`). | Cactoos/Takes READMEs make strong claims ("zero null", "zero deps") that are goals, not always code-true. | Marketing + contract overlap. | **[SHOULD]** verify README claims against code; treat them as a spec to enforce, not a fact. |
| **AOP** | Book embraces it for cross-cutting concerns (`B6.11`, `B11.4`); `Adt.2` rejects behavior annotations. | jcabi-aspects used widely in s3auth/requs; `Adt.21` wants visible `new FooThatRetries(new Foo())`. | Sources disagree. | **[SHOULD]** prefer an explicit decorator for retry/logging; **[NICE]** an aspect only when weaving is unavoidable (`Adt.65`). |
| **Big test classes** | 250-LOC limit applies to tests (`B1.21`, `B8.10`). | `Atq.37` says a 5,000-line `FooTest` is fine (a container of scripts); real suites are large. | One test class per live class. | **[SHOULD]** keep tests focused per behavior; **[NICE]** a long test class is acceptable if it maps 1:1 to one production class. |


