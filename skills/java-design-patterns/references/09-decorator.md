# 09. Decorator

> **Category**: Structural  
> **Intent**: Attach additional responsibilities to an object dynamically. Decorators provide a more flexible alternative to subclassing for extending functionality.

## Usage

The decorator adds functionality by wrapping an object while keeping the interface unchanged. A decorator implements the same interface as the component and holds a component instance; on invocation, it first executes its own enhancement logic and then delegates to the wrapped object. Multiple decorators can be layered, and the order affects the final result.

1. Define the component interface.
2. Implement the base component.
3. Define a decorator base class that implements the component interface and holds a component.
4. Each concrete decorator adds behavior before or after delegation.
5. The client layers decorators as needed.

## Use Cases

- Java IO streams: buffering, compression, encryption.
- Adding logging, caching, rate limiting, retries, or permissions to an interface.
- Dynamically combining functionality at runtime.

## Pros

- More flexible than inheritance; functionality can be layered dynamically.
- Each decorator has a single responsibility.
- Avoids generating a subclass for every combination.

## Cons

- Produces many small objects.
- Decorator order matters, making problems harder to diagnose.
- Multiple wrapping layers add call overhead.

## Java Example

```java
// Component interface: description and price of a coffee
interface Coffee {
    // Return the coffee description
    String description();

    // Return the coffee price
    double cost();
}

// Base component: Americano
class SimpleCoffee implements Coffee {
    // Return the base description
    @Override
    public String description() {
        return "Americano";
    }

    // Return the base price
    @Override
    public double cost() {
        return 18.0;
    }
}

// Decorator base class: holds the decorated coffee
abstract class CoffeeDecorator implements Coffee {
    protected final Coffee coffee;

    // Inject the coffee object to decorate
    protected CoffeeDecorator(Coffee coffee) {
        this.coffee = coffee;
    }
}

// Milk decorator
class MilkDecorator extends CoffeeDecorator {
    // Create a milk decorator
    public MilkDecorator(Coffee coffee) {
        super(coffee);
    }

    // Append milk to the original description
    @Override
    public String description() {
        return coffee.description() + " + milk";
    }

    // Add 3 CNY to the original price
    @Override
    public double cost() {
        return coffee.cost() + 3.0;
    }
}

// Sugar decorator
class SugarDecorator extends CoffeeDecorator {
    // Create a sugar decorator
    public SugarDecorator(Coffee coffee) {
        super(coffee);
    }

    // Append sugar to the original description
    @Override
    public String description() {
        return coffee.description() + " + sugar";
    }

    // Add 1 CNY to the original price
    @Override
    public double cost() {
        return coffee.cost() + 1.0;
    }
}

public class DecoratorDemo {
    // Entry point: layer the milk and sugar decorators
    public static void main(String[] args) {
        Coffee coffee = new SugarDecorator(new MilkDecorator(new SimpleCoffee()));

        System.out.println(coffee.description());
        System.out.printf("Total: %.2f CNY%n", coffee.cost());
    }
}
```

---

[Back to skill overview](../SKILL.md)
