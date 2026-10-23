---
name: java-design-patterns
description: A complete reference to the 23 GoF design patterns in Java, covering usage, applicable scenarios, pros and cons, and compilable, runnable Java examples. Use this skill whenever the user mentions design patterns, singleton, factory method, abstract factory, builder, prototype, adapter, bridge, composite, decorator, facade, flyweight, proxy, chain of responsibility, command, interpreter, iterator, mediator, memento, observer, state, strategy, template method, or visitor, or asks for refactoring, decoupling, extension-point design, pattern selection, or code review in a Java project. Even when the user does not explicitly say "design pattern", consult the corresponding reference document whenever the problem involves object creation, structural composition, behavioral decoupling, algorithm replacement, or state transitions.
---

# Java Design Patterns

This skill provides a complete English reference for the 23 GoF design patterns. Each pattern has its own document, ordered as follows: intent, usage, use cases, pros, cons, and a complete Java example.

## Decide First, Then Read the Docs

Design patterns are tools for handling points of change, not code decoration. Before opening a document, confirm:

1. Where is the point of change: object creation, structural composition, or responsibilities and communication between objects?
2. Which parts must stay stable, and which parts must vary independently?
3. Is there a simpler approach: direct `new`, a plain interface, composition, a functional interface, or no abstraction at all?

Once you have identified candidate patterns, read only the relevant documents. Do not apply several patterns at once.

## Pattern Index

### Creational

- [01 Singleton](references/01-singleton.md): the number of instances must be strictly controlled, for example a configuration center, connection pool, or logger.
- [02 Factory Method](references/02-factory-method.md): creation logic must be deferred to subclasses, or different products must be returned by type.
- [03 Abstract Factory](references/03-abstract-factory.md): a family of related products must be created and guaranteed to be used together.
- [04 Builder](references/04-builder.md): there are many constructor parameters and optional parameters, and an immutable object must be built step by step.
- [05 Prototype](references/05-prototype.md): object creation is expensive, or an existing object must be copied with a few differences.

### Structural

- [06 Adapter](references/06-adapter.md): an existing interface is incompatible with the interface the client expects.
- [07 Bridge](references/07-bridge.md): two dimensions both need independent extension, avoiding M×N subclasses.
- [08 Composite](references/08-composite.md): leaf nodes and container nodes in a tree structure must be handled uniformly.
- [09 Decorator](references/09-decorator.md): functionality must be layered dynamically at runtime instead of generating many subclasses through inheritance.
- [10 Facade](references/10-facade.md): subsystem calls are complex, and a simple, unified high-level entry point is needed.
- [11 Flyweight](references/11-flyweight.md): there are many duplicate objects, and most of their state can be shared.
- [12 Proxy](references/12-proxy.md): access must be controlled, loading delayed, or logging, permissions, transactions, and similar logic added before and after calls.

### Behavioral

- [13 Chain of Responsibility](references/13-chain-of-responsibility.md): multiple handlers try to handle a request in order, or the handling flow must be assembled dynamically.
- [14 Command](references/14-command.md): a request must be encapsulated as an object to support queuing, logging, undo, or macro commands.
- [15 Interpreter](references/15-interpreter.md): a simple grammar or DSL must be interpreted, and the grammar rules are relatively stable.
- [16 Iterator](references/16-iterator.md): aggregate objects must be traversed uniformly while hiding their internal structure.
- [17 Mediator](references/17-mediator.md): multiple objects form a mesh of dependencies, and interactions must be centralized in a mediator.
- [18 Memento](references/18-memento.md): object snapshots must be saved and restored, for example for undo, rollback, or save points.
- [19 Observer](references/19-observer.md): one object's state change must notify multiple dependents.
- [20 State](references/20-state.md): an object's behavior changes with its internal state, and there are explicit state transitions.
- [21 Strategy](references/21-strategy.md): multiple algorithms are interchangeable, and the client must choose one at runtime.
- [22 Template Method](references/22-template-method.md): the process skeleton is fixed, and only a few steps need subclass implementations.
- [23 Visitor](references/23-visitor.md): the object structure is stable, but operations that act on it must keep being added.

## Workflow

1. **Identify the point of change**: state in one sentence where the current code hurts and what is most likely to change in the future.
2. **Pick 1–2 candidates**: choose the closest pattern from the index and read its "Usage" and "Use Cases" sections.
3. **Make the trade-off**: compare the "Pros/Cons" and explain why you chose it, and why you did not choose a simpler solution.
4. **Rewrite the code**: adapt interfaces, naming, and exception handling based on the reference example, follow the project's existing style, and do not copy the example's business domain.
5. **Verify**: if the environment allows, compile and run the example; at minimum check generics, null values, concurrency, resource release, and state-transition boundaries.
6. **Explain**: output the pattern name, the problem it solves, the cost it introduces, and the alternatives.

## Common Comparisons

- **Factory Method vs. Abstract Factory**: the former usually creates one product, while the latter creates a family of related products.
- **Adapter vs. Bridge**: an adapter converts an existing interface after the fact; a bridge separates two independently varying dimensions at design time.
- **Decorator vs. Proxy**: a decorator emphasizes layering functionality, while a proxy emphasizes controlling access; both implement the same interface and hold the target object.
- **Strategy vs. State**: the strategy is chosen by the client, and algorithms usually do not switch automatically; state is driven by internal events, and states do transition.
- **Composition vs. Inheritance**: prefer composition when behavior must be combined dynamically; inheritance suits a stable "is-a" relationship.
- **Template Method vs. Strategy**: template method fixes the flow through inheritance, while strategy replaces the entire algorithm through composition.

## Output Requirements

- Explain in English; code comments in English.
- Java examples prefer Java 17+ syntax, stay compilable in a single file, and do not depend on third-party frameworks.
- Do not add abstraction just to show off a pattern; if the direct approach is clearer, explicitly state "no pattern is needed here".
- When singleton, flyweight, observer, proxy, and similar patterns are involved, call out thread safety, memory leaks, lifecycle, and performance costs.
- When the user asks for a comparison, give applicable conditions, costs, and a minimal example rather than only definitions.

## Reference Documents

All 23 documents are under `references/`. Each contains usage, use cases, pros, cons, and a complete Java example. Read the corresponding document from the index first; do not load all documents at once.
