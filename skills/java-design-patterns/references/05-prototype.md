# 05. Prototype

> **Category**: Creational  
> **Intent**: Specify the kinds of objects to create using a prototypical instance, and create new objects by copying this prototype.

## Usage

The prototype pattern creates new objects by copying an existing object instead of going through the construction process again. Implement it by making the class implement `Cloneable` and override `clone()`; call `super.clone()` for a shallow copy first, then manually deep-copy reference-type fields. When the object structure is complex, a copy constructor or serialization is often more controllable than `clone()`.

1. Make the prototype class implement `Cloneable`.
2. Override `clone()` and call `super.clone()`.
3. Deep-copy mutable reference fields.
4. Let the client copy the prototype and then modify the differences.
5. Watch out for circular references and immutable objects, and avoid sharing mutable state.

## Use Cases

- Object creation is expensive, for example when it requires database queries, computation, or remote calls.
- A snapshot of an object at a particular moment must be saved.
- Many objects differ in only a few attributes.

## Pros

- Copying is usually faster than creating from scratch.
- Prototypes can be added or removed at runtime.
- Simplifies the creation of similar objects.

## Cons

- Deep copying is complex to implement, and circular references are hard to handle.
- `Cloneable` is a marker interface and is easy to misuse by design.
- Every class must implement cloning logic, raising maintenance costs.

## Java Example

```java
import java.util.ArrayList;
import java.util.List;

// Prototype class: implements Cloneable and supports copying itself
class Resume implements Cloneable {
    private String name;
    private List<String> skills;

    // Create a resume, making a defensive copy of the skill list
    public Resume(String name, List<String> skills) {
        this.name = name;
        this.skills = new ArrayList<>(skills);
    }

    // Copy the resume: shallow copy first, then copy reference-type fields separately
    @Override
    public Resume clone() {
        try {
            Resume copy = (Resume) super.clone();
            // List is a mutable reference, so a new list is required to avoid sharing the same data
            copy.skills = new ArrayList<>(this.skills);
            return copy;
        } catch (CloneNotSupportedException e) {
            throw new AssertionError("Resume does not implement Cloneable", e);
        }
    }

    // Append a skill to the resume
    public void addSkill(String skill) {
        skills.add(skill);
    }

    // Print the resume contents for comparing the original and the copy
    public void print() {
        System.out.println(name + "'s skills: " + skills);
    }
}

public class PrototypeDemo {
    // Entry point: copy the prototype, modify the copy, and verify they do not affect each other
    public static void main(String[] args) {
        Resume original = new Resume("Zhang San", List.of("Java", "MySQL"));
        Resume copy = original.clone();

        copy.addSkill("Redis");

        original.print();
        copy.print();
    }
}
```

---

[Back to skill overview](../SKILL.md)
