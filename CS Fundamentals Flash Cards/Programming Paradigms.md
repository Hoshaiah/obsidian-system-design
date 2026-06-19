---
tags:
  - flashcards/paradigms
---

# Programming Paradigms

OOP, functional, declarative — and the design principles that travel between them.

---

What are the four pillars of OOP and what do they each buy you?
?
**Quick:** Encapsulation (hide state), inheritance (reuse type), polymorphism (one interface, many implementations), abstraction (model the essentials).<br>
**Deeper:** Encapsulation enforces invariants by limiting who can mutate state. Inheritance is overused — "composition over inheritance" is now standard advice because deep hierarchies are fragile. Polymorphism (via interfaces/traits) is the durable win: substitutability enables testing (mocks), strategies, and dependency injection. Modern OOP often looks more like interface-driven design with composition.

What's the difference between an abstract class and an interface, and when do you use each?
?
**Quick:** Abstract classes can hold state and partial implementation; interfaces specify capability without state (mostly).<br>
**Deeper:** Use interfaces when you want a capability contract orthogonal to type hierarchy (Iterable, Comparable). Use abstract classes when you have shared implementation but want to forbid direct instantiation. Modern languages blur this — Java 8+ interfaces have default methods, Rust traits have default impls — so the choice is increasingly about whether state and constructor invariants are needed.

What does the SOLID acronym stand for?
?
**Quick:** Single Responsibility, Open/Closed, Liskov Substitution, Interface Segregation, Dependency Inversion.<br>
**Deeper:** SRP: one reason to change. OCP: extend without modifying. LSP: subtypes must be substitutable for parents (violations cause weird bugs — Square inheriting Rectangle is the classic example). ISP: clients shouldn't depend on methods they don't use. DIP: depend on abstractions, not concretions. SOLID is most useful when applied at architecture seams (modules, services), not religiously inside every class.

What is the Liskov Substitution Principle and what's a common violation?
?
**Quick:** Anywhere a parent type is expected, a subclass instance should work without surprises; common violations strengthen preconditions or weaken postconditions.<br>
**Deeper:** A Square-as-Rectangle subclass that forbids unequal width/height violates LSP because code expecting Rectangle's contract breaks. Other violations: subclasses that throw NotSupportedException, mutate inherited invariants, or add new exceptions. LSP failures are why deep inheritance hierarchies become bug factories.

What does "composition over inheritance" mean?
?
**Quick:** Build features by combining objects (has-a) rather than extending types (is-a); more flexible and avoids fragile base classes.<br>
**Deeper:** Inheritance binds you to the parent's lifecycle, internals, and evolution. Composition lets you swap behaviors at runtime, mix capabilities (think Go's embedding, Rust's traits), and test components in isolation. The "decorator" and "strategy" patterns are composition wins. Most modern frameworks default to composition (React hooks, dependency injection, middleware chains).

What's a pure function and why does it matter for functional programming?
?
**Quick:** Same input always yields same output, with no side effects; enables equational reasoning and easy testing.<br>
**Deeper:** Pure functions are referentially transparent, so you can cache results (memoization), parallelize freely, and reason locally. Side effects must live at the system boundary (IO monads in Haskell, "imperative shell, functional core" patterns elsewhere). The discipline pays off even in OOP codebases: pure helpers reduce bugs and speed up tests.

What are higher-order functions and why are they powerful?
?
**Quick:** Functions that take or return other functions; enable abstraction over behavior, not just data.<br>
**Deeper:** Map/filter/reduce, decorators, callbacks, comparators — all higher-order. They let you parameterize control flow (strategy pattern without the boilerplate) and build pipelines (streams, observables). Languages without first-class functions resort to verbose interfaces (Java pre-8 anonymous classes). Closures (functions capturing surrounding state) are the runtime mechanism that makes this all work.

What is immutability and why is it useful for concurrent code?
?
**Quick:** Immutable objects can't change after construction; safe to share across threads without synchronization.<br>
**Deeper:** Immutability eliminates entire classes of bugs: no race conditions on the data, no defensive copies, easier reasoning. Persistent data structures (Clojure, Immer, Scala) give "modifications" by creating new versions sharing structure with the old. Cost: more allocations, more GC pressure — but the tradeoff usually wins for application-level code.

What's the difference between imperative and declarative programming?
?
**Quick:** Imperative says how to do something step by step; declarative says what you want without specifying how.<br>
**Deeper:** SQL, regex, HTML, React's JSX, Terraform — all declarative. The runtime/compiler picks the strategy. Tradeoff: declarative is more composable and easier to reason about correctness, but you lose control over performance. A SQL query relies on the optimizer; if it picks wrong, you may have to write hints or restructure. Most modern systems mix paradigms intentionally.

What is dependency injection and what problem does it solve?
?
**Quick:** Provide a class's dependencies from outside instead of constructing them internally; enables testing and configurability.<br>
**Deeper:** Without DI, a class is coupled to specific implementations (hard to test, hard to swap). DI inverts the relationship: the class declares what it needs, a container or test wires it. Pattern works regardless of language. Heavyweight DI frameworks (Spring, Guice) automate wiring but can obscure flow; many modern codebases prefer "poor man's DI" — just constructor injection.

What is the open/closed principle and how do you achieve it in practice?
?
**Quick:** Code should be open for extension but closed for modification; achieve via polymorphism, plugins, or configuration.<br>
**Deeper:** A switch statement that grows with every new type violates OCP; refactor to a polymorphic dispatch (strategy pattern, interface implementations). Plugin architectures, event hooks, and visitor patterns all serve OCP. Goal: adding a new case shouldn't require modifying core logic. Beware over-application — premature abstraction is its own cost.

What's the difference between procedural and object-oriented programming?
?
**Quick:** Procedural organizes code as procedures operating on data; OOP bundles data and behavior together.<br>
**Deeper:** Procedural (C, early Pascal) scales poorly when data and behavior need to evolve together — every function knows the struct layout. OOP encapsulates this. Modern "data-oriented design" pushes back: bundling data and behavior can hurt cache performance and parallelism. The pendulum keeps swinging; healthy codebases pick the right paradigm per layer.
