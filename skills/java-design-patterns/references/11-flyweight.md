# 11. Flyweight

> **Category**: Structural  
> **Intent**: Use sharing to support large numbers of fine-grained objects efficiently.

## Usage

The flyweight pattern splits an object's state into intrinsic state and extrinsic state. Intrinsic state can be shared; it is stored in the flyweight object and kept immutable. Extrinsic state varies with the usage context and is stored or passed in by the client. A factory caches objects with the same intrinsic state to reuse them, thereby reducing the number of objects.

1. Distinguish intrinsic state from extrinsic state.
2. Define the flyweight object to store only intrinsic state and make it immutable.
3. Create a flyweight factory that caches flyweights in a Map.
4. The client keeps extrinsic state and passes it to the flyweight when needed.
5. Pay attention to concurrent access and cache cleanup.

## Use Cases

- Character styles in a text editor, chess pieces on a board, map markers.
- Memory pressure caused by a large number of duplicate objects.
- Most of an object's state can be shared.

## Pros

- Significantly reduces the number of objects and memory usage.
- The shared structure is clear and easy to manage uniformly.
- Extrinsic state is flexible and does not pollute shared objects.

## Cons

- Adds factory and lookup overhead.
- The intrinsic/extrinsic split requires careful design.
- Shared objects are immutable, limiting flexibility.

## Java Example

```java
import java.util.HashMap;
import java.util.Map;

// Flyweight: the intrinsic state of a tree, shareable and immutable
class TreeType {
    private final String name;
    private final String color;

    // Create a tree type; intrinsic state is initialized only in the factory
    public TreeType(String name, String color) {
        this.name = name;
        this.color = color;
    }

    // Draw the tree at the position passed in as extrinsic state
    public void draw(int x, int y) {
        System.out.println("Drawing a " + color + " " + name + " at (" + x + ", " + y + ")");
    }
}

// Flyweight factory: create only one object for the same intrinsic state
class TreeFactory {
    private static final Map<String, TreeType> CACHE = new HashMap<>();

    // Get a flyweight by name and color, creating and caching it when absent
    public static TreeType getTreeType(String name, String color) {
        String key = name + "|" + color;
        return CACHE.computeIfAbsent(key, k -> new TreeType(name, color));
    }
}

// Extrinsic state: position, kept by the client
class Tree {
    private final int x;
    private final int y;
    private final TreeType type;

    // Create a tree, combining the extrinsic position with the shared tree type
    public Tree(int x, int y, TreeType type) {
        this.x = x;
        this.y = y;
        this.type = type;
    }

    // Draw at the current position using the shared tree type
    public void draw() {
        type.draw(x, y);
    }
}

public class FlyweightDemo {
    // Entry point: many trees share a small number of tree types
    public static void main(String[] args) {
        TreeType ginkgo = TreeFactory.getTreeType("Ginkgo", "green");
        TreeType another = TreeFactory.getTreeType("Ginkgo", "green");

        System.out.println("Same flyweight object: " + (ginkgo == another));

        for (int i = 0; i < 3; i++) {
            new Tree(i * 10, i * 20, ginkgo).draw();
        }
    }
}
```

---

[Back to skill overview](../SKILL.md)
