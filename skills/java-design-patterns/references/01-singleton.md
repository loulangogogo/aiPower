# 01. Singleton Pattern

> **Category**: Creational  
> **Intent**: Ensure that a class has only one instance and provide a global access point to it.

## Usage

The singleton class is responsible for creating, storing, and exposing the sole instance. Implement it by making the constructor private, creating a static instance inside the class, and returning it through a static method. Lazy loading can use a static nested class; when the singleton must be protected from reflection and deserialization, an enum is the safer choice.

1. Make the constructor private so external code cannot call `new`.
2. Create and store the sole instance inside the class.
3. Provide a static access method that always returns that instance.
4. Choose eager initialization, a static nested class, or an enum according to the scenario, and handle concurrency and reflection.

## Use Cases

- Global configuration, application-level caches, loggers.
- Resources that need centralized management, such as connection pools and thread pools.
- Hardware or system resources whose instance count must be strictly controlled.

## Pros

- Controls the number of instances and reduces the overhead of repeated creation.
- Provides a unified access point with centralized state.
- Supports lazy initialization, creating the instance on demand.

## Cons

- Global state makes dependencies implicit and hard to replace in unit tests.
- Reflection, deserialization, and multi-threaded initialization all have pitfalls.
- Abuse hides module-boundary problems.

## Java Example

```java
public class SingletonDemo {
    // The static nested class holds the sole instance, initialized on first access, inherently thread-safe
    private static class Holder {
        private static final SingletonDemo INSTANCE = new SingletonDemo();
    }

    // Private constructor prevents external code from creating instances directly
    private SingletonDemo() {
        // Prevent duplicate instances created through reflection
        if (Holder.INSTANCE != null) {
            throw new IllegalStateException("Singleton has already been initialized");
        }
    }

    // Return the globally unique instance
    public static SingletonDemo getInstance() {
        return Holder.INSTANCE;
    }

    // Business method demonstrating the singleton
    public void log(String message) {
        System.out.println("[LOG] " + message);
    }

    // Entry point: verify that multiple calls return the same object
    public static void main(String[] args) {
        SingletonDemo first = SingletonDemo.getInstance();
        SingletonDemo second = SingletonDemo.getInstance();

        System.out.println("Same instance: " + (first == second));
        first.log("Application started");
    }
}
```

---

[Back to skill overview](../SKILL.md)
