# 08. Composite

> **Category**: Structural  
> **Intent**: Compose objects into tree structures to represent part-whole hierarchies, so that clients can treat individual objects and compositions of objects uniformly.

## Usage

The composite pattern organizes objects into a tree structure, where leaf nodes and container nodes implement the same interface. A container node holds a list of child nodes and recursively delegates operations to them. The client does not need to determine whether the current node is a leaf or a container; it simply calls the interface uniformly.

1. Define a unified component interface.
2. Leaf nodes implement the basic operations.
3. Container nodes implement the same interface and maintain a collection of child nodes.
4. Operations on containers recursively call their child nodes.
5. The client operates on the entire tree through the component interface.

## Use Cases

- Files and directories, organizational structures, menu trees.
- UI containers and controls.
- Statistics, rendering, or validation that must be applied uniformly to an entire tree.

## Pros

- Treats individual objects and compositions uniformly.
- The tree structure is natural to express, and recursion keeps the implementation concise.
- Adding new node types is relatively easy.

## Cons

- The interface is too general, making it hard to restrict a container to a certain node type.
- Deep recursion may cause a stack overflow.
- Debugging and troubleshooting recursive logic is harder.

## Java Example

```java
import java.util.ArrayList;
import java.util.List;

// Component interface: both leaf nodes and container nodes implement it
interface FileSystemNode {
    // Print the node content; indent is the current indentation
    void show(String indent);
}

// Leaf node: a regular file
class FileNode implements FileSystemNode {
    private final String name;

    // Create a file node
    public FileNode(String name) {
        this.name = name;
    }

    // Print the file name
    @Override
    public void show(String indent) {
        System.out.println(indent + "- " + name);
    }
}

// Container node: a directory holding a list of child nodes
class DirectoryNode implements FileSystemNode {
    private final String name;
    private final List<FileSystemNode> children = new ArrayList<>();

    // Create a directory node
    public DirectoryNode(String name) {
        this.name = name;
    }

    // Add a file or subdirectory to the directory
    public void add(FileSystemNode node) {
        children.add(node);
    }

    // Print the directory name and recursively print all child nodes
    @Override
    public void show(String indent) {
        System.out.println(indent + "+ " + name + "/");
        for (FileSystemNode child : children) {
            child.show(indent + "  ");
        }
    }
}

public class CompositeDemo {
    // Entry point: build a directory tree and render it uniformly
    public static void main(String[] args) {
        DirectoryNode root = new DirectoryNode("project");
        DirectoryNode src = new DirectoryNode("src");

        src.add(new FileNode("Main.java"));
        src.add(new FileNode("Utils.java"));
        root.add(src);
        root.add(new FileNode("README.md"));

        root.show("");
    }
}
```

---

[Back to skill overview](../SKILL.md)
