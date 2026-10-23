# 03. Abstract Factory

> **Category**: Creational  
> **Intent**: Provide an interface for creating a series of related or interdependent objects without specifying their concrete classes.

## Usage

The abstract factory targets a "product family": one factory is responsible for creating a group of related products. The client depends only on the abstract factory and abstract products, and switching the entire implementation requires only replacing the factory. Its difference from the factory method is that the factory method usually creates only one product, while the abstract factory creates a family of products.

1. Define an interface for each product kind.
2. Define the abstract factory and declare creation methods for a family of products.
3. Implement a concrete factory for each product family, ensuring products of the same family are used together.
4. The client uses only the abstract factory and abstract products.

## Use Cases

- Cross-platform UI: buttons, checkboxes, and menus for Windows and macOS.
- Multiple themes, multiple database dialects, and clients for multiple environments.
- Objects of the same family must be used together and must not be mixed.

## Pros

- Guarantees consistency among objects within a product family.
- Decouples the client from concrete implementations.
- Switching product families requires only replacing the factory.

## Cons

- Adding a product kind requires modifying all factories, violating the open-closed principle.
- The number of classes increases noticeably.
- The abstraction layer is heavy, making small projects prone to over-engineering.

## Java Example

```java
// Button interface in the product family
interface Button {
    // Render the button
    void render();
}

// Checkbox interface in the product family
interface Checkbox {
    // Render the checkbox
    void render();
}

// Windows-style button
class WindowsButton implements Button {
    // Output the rendering result of the Windows button
    @Override
    public void render() {
        System.out.println("Rendering a Windows button");
    }
}

// Windows-style checkbox
class WindowsCheckbox implements Checkbox {
    // Output the rendering result of the Windows checkbox
    @Override
    public void render() {
        System.out.println("Rendering a Windows checkbox");
    }
}

// macOS-style button
class MacButton implements Button {
    // Output the rendering result of the macOS button
    @Override
    public void render() {
        System.out.println("Rendering a macOS button");
    }
}

// macOS-style checkbox
class MacCheckbox implements Checkbox {
    // Output the rendering result of the macOS checkbox
    @Override
    public void render() {
        System.out.println("Rendering a macOS checkbox");
    }
}

// Abstract factory: declare methods for creating a family of UI components
interface UiFactory {
    // Create a button
    Button createButton();

    // Create a checkbox
    Checkbox createCheckbox();
}

// Windows product-family factory
class WindowsFactory implements UiFactory {
    // Create a Windows button
    @Override
    public Button createButton() {
        return new WindowsButton();
    }

    // Create a Windows checkbox
    @Override
    public Checkbox createCheckbox() {
        return new WindowsCheckbox();
    }
}

// macOS product-family factory
class MacFactory implements UiFactory {
    // Create a macOS button
    @Override
    public Button createButton() {
        return new MacButton();
    }

    // Create a macOS checkbox
    @Override
    public Checkbox createCheckbox() {
        return new MacCheckbox();
    }
}

public class AbstractFactoryDemo {
    // Entry point: use the same factory to create components of the same family
    public static void main(String[] args) {
        UiFactory factory = new MacFactory();
        Button button = factory.createButton();
        Checkbox checkbox = factory.createCheckbox();

        button.render();
        checkbox.render();
    }
}
```

---

[Back to skill overview](../SKILL.md)
