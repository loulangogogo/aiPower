# 23. Visitor

> **Category**: Behavioral  
> **Intent**: Represent an operation to be performed on the elements of an object structure, letting you define new operations without changing the classes of the elements on which they operate.

## Usage

The visitor pattern separates operations on an object structure from the element classes. The element interface declares `accept(visitor)`, and each element calls the visitor's corresponding `visit()` method in `accept()`, forming double dispatch. Adding an operation requires only a new visitor without modifying elements; but adding an element type requires modifying all visitors.

1. Define the visitor interface and declare a `visit()` overload for each element type.
2. Define the element interface and declare `accept(visitor)`.
3. Each element calls `visitor.visit(this)` in `accept()`.
4. Concrete visitors implement the operations.
5. The object structure traverses its elements and accepts visitors.

## Use Cases

- AST/syntax-tree traversal, compiler checks.
- Report export, multiple statistics over an object structure.
- A stable object structure with frequently added operations.

## Pros

- Adding operations is easy and follows the open-closed principle.
- Related operations are centralized in one visitor.
- Element classes stay stable with clear responsibilities.

## Cons

- Adding an element type requires modifying all visitors.
- Visitors may require elements to expose internal state.
- Double dispatch is not intuitive and has a high learning cost.

## Java Example

```java
// Visitor interface: declares one visit overload for each element type
interface ShapeVisitor {
    // Visit a circle
    void visit(Circle circle);

    // Visit a rectangle
    void visit(Rectangle rectangle);
}

// Element interface: accepts a visitor
interface Shape {
    // Hand itself to the visitor for processing
    void accept(ShapeVisitor visitor);
}

// Concrete element: circle
class Circle implements Shape {
    private final double radius;

    // Create a circle with the specified radius
    public Circle(double radius) {
        this.radius = radius;
    }

    // Return the radius for the visitor to read
    public double getRadius() {
        return radius;
    }

    // Call the visitor's circle-handling method
    @Override
    public void accept(ShapeVisitor visitor) {
        visitor.visit(this);
    }
}

// Concrete element: rectangle
class Rectangle implements Shape {
    private final double width;
    private final double height;

    // Create a rectangle with the specified width and height
    public Rectangle(double width, double height) {
        this.width = width;
        this.height = height;
    }

    // Return the width for the visitor to read
    public double getWidth() {
        return width;
    }

    // Return the height for the visitor to read
    public double getHeight() {
        return height;
    }

    // Call the visitor's rectangle-handling method
    @Override
    public void accept(ShapeVisitor visitor) {
        visitor.visit(this);
    }
}

// Visitor: compute the total area
class AreaVisitor implements ShapeVisitor {
    private double total;

    // Accumulate the circle area
    @Override
    public void visit(Circle circle) {
        total += Math.PI * circle.getRadius() * circle.getRadius();
    }

    // Accumulate the rectangle area
    @Override
    public void visit(Rectangle rectangle) {
        total += rectangle.getWidth() * rectangle.getHeight();
    }

    // Return the accumulated area
    public double getTotal() {
        return total;
    }
}

// Visitor: export shape descriptions
class DescriptionVisitor implements ShapeVisitor {
    // Print the circle description
    @Override
    public void visit(Circle circle) {
        System.out.printf("Circle, radius %.2f%n", circle.getRadius());
    }

    // Print the rectangle description
    @Override
    public void visit(Rectangle rectangle) {
        System.out.printf("Rectangle, width %.2f, height %.2f%n", rectangle.getWidth(), rectangle.getHeight());
    }
}

public class VisitorDemo {
    // Entry point: let the same set of elements accept two kinds of visitors
    public static void main(String[] args) {
        Shape[] shapes = {
                new Circle(2),
                new Rectangle(3, 4)
        };

        AreaVisitor areaVisitor = new AreaVisitor();
        DescriptionVisitor descriptionVisitor = new DescriptionVisitor();

        for (Shape shape : shapes) {
            shape.accept(areaVisitor);
            shape.accept(descriptionVisitor);
        }

        System.out.printf("Total area: %.2f%n", areaVisitor.getTotal());
    }
}
```

---

[Back to skill overview](../SKILL.md)
