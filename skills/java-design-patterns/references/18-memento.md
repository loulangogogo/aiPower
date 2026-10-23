# 18. Memento

> **Category**: Behavioral  
> **Intent**: Without violating encapsulation, capture and externalize an object's internal state so that the object can be restored to this state later.

## Usage

The memento pattern saves an object's internal state externally so it can be restored later. The originator is responsible for creating and restoring snapshots; the memento only stores state and is read-only to the outside; the caretaker stores snapshots but does not modify their contents. When there are many snapshots, consider memory usage, and limit history length or use incremental snapshots.

1. Define the memento class, storing state and providing read-only access.
2. The originator provides `save()` and `restore()`.
3. The caretaker stores mementos in a stack or list.
4. Hand the memento back to the originator for restoration when needed.
5. Control the history length and snapshot size.

## Use Cases

- Editor undo/redo, form drafts, game saves.
- Transaction rollback, process rollback.
- Saving an object's historical states.

## Pros

- Saves state without violating encapsulation.
- Restoration logic is simple.
- History management is centralized in the caretaker.

## Cons

- Snapshots consume memory.
- Frequent snapshots hurt performance.
- The caretaker lifecycle must be managed.

## Java Example

```java
import java.util.ArrayDeque;
import java.util.Deque;

// Memento: stores state only and provides no mutators
final class EditorMemento {
    private final String content;

    // Create a snapshot and store the content
    EditorMemento(String content) {
        this.content = content;
    }

    // Return the content in the snapshot
    String getContent() {
        return content;
    }
}

// Originator: responsible for creating and restoring snapshots
class Editor {
    private String content = "";

    // Append typed content
    public void type(String text) {
        content += text;
    }

    // Return the current content
    public String getContent() {
        return content;
    }

    // Create a snapshot of the current state
    public EditorMemento save() {
        return new EditorMemento(content);
    }

    // Restore the content from a snapshot
    public void restore(EditorMemento memento) {
        this.content = memento.getContent();
    }
}

// Caretaker: only stores snapshots and does not care about their contents
class History {
    private final Deque<EditorMemento> stack = new ArrayDeque<>();

    // Save a snapshot
    public void push(EditorMemento memento) {
        stack.push(memento);
    }

    // Take out the most recent snapshot
    public EditorMemento pop() {
        return stack.pop();
    }
}

public class MementoDemo {
    // Entry point: edit text, save snapshots, and undo step by step
    public static void main(String[] args) {
        Editor editor = new Editor();
        History history = new History();

        editor.type("First version");
        history.push(editor.save());

        editor.type(", second version");
        history.push(editor.save());

        editor.type(", third version");
        System.out.println("Current content: " + editor.getContent());

        editor.restore(history.pop());
        System.out.println("After one undo: " + editor.getContent());

        editor.restore(history.pop());
        System.out.println("After another undo: " + editor.getContent());
    }
}
```

---

[Back to skill overview](../SKILL.md)
