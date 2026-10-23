# 22. Template Method

> **Category**: Behavioral  
> **Intent**: Define the skeleton of an algorithm in an operation, deferring some steps to subclasses, so that subclasses can redefine certain steps without changing the algorithm's structure.

## Usage

The template method defines the algorithm skeleton in an abstract class and leaves the variable steps to subclasses. The skeleton method is usually declared `final` to prevent subclasses from breaking the flow; variable steps can be abstract methods or hook methods with default implementations. It relies on inheritance and suits scenarios where the flow is stable and the step differences are clear.

1. Define a `final` template method in the abstract class.
2. Implement fixed steps as private methods.
3. Declare variable steps as abstract methods or hook methods.
4. Subclasses implement the concrete steps.
5. The client calls the template method.

## Use Cases

- Framework lifecycles, data import/export, test base classes.
- Servlet's `service()` calling `doGet()`/`doPost()`.
- `JdbcTemplate`, `InputStream.read()`, and similar APIs.

## Pros

- Reuses the algorithm skeleton and reduces duplicate code.
- The flow is controlled; subclasses can change only designated steps.
- Follows the open-closed principle.

## Cons

- Relies on inheritance, limiting flexibility.
- Changes to the parent class affect all subclasses.
- Too many steps make the inheritance hierarchy hard to maintain.

## Java Example

```java
// Abstract class: defines the fixed flow for brewing a beverage
abstract class Beverage {
    // Template method: fixes the brewing order; subclasses cannot override it
    public final void prepare() {
        boilWater();
        brew();
        pourInCup();
        addCondiments();
        hook();
    }

    // Variable step: brew the main ingredient
    protected abstract void brew();

    // Variable step: add condiments
    protected abstract void addCondiments();

    // Fixed step: boil water
    private void boilWater() {
        System.out.println("Boiling water");
    }

    // Fixed step: pour into the cup
    private void pourInCup() {
        System.out.println("Pouring into cup");
    }

    // Hook method: subclasses may override it optionally
    protected void hook() {
    }
}

// Concrete class: coffee
class Coffee extends Beverage {
    // Brew coffee grounds
    @Override
    protected void brew() {
        System.out.println("Brewing coffee grounds");
    }

    // Add sugar and milk
    @Override
    protected void addCondiments() {
        System.out.println("Adding sugar and milk");
    }
}

// Concrete class: tea
class Tea extends Beverage {
    // Steep tea leaves
    @Override
    protected void brew() {
        System.out.println("Steeping tea leaves");
    }

    // Add lemon
    @Override
    protected void addCondiments() {
        System.out.println("Adding lemon");
    }

    // Override the hook to print an extra reminder
    @Override
    protected void hook() {
        System.out.println("Note: drink the tea while it is hot");
    }
}

public class TemplateMethodDemo {
    // Entry point: brew coffee and tea separately
    public static void main(String[] args) {
        Beverage coffee = new Coffee();
        coffee.prepare();

        System.out.println("---");

        Beverage tea = new Tea();
        tea.prepare();
    }
}
```

---

[Back to skill overview](../SKILL.md)
