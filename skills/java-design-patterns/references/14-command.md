# 14. Command

> **Category**: Behavioral  
> **Intent**: Encapsulate a request as an object, thereby letting you parameterize clients with different requests, queue or log requests, and support undoable operations.

## Usage

The command pattern encapsulates a request as an object. The command interface declares `execute()`, and a concrete command holds a receiver and calls its business methods; the invoker holds only the command and does not care who the receiver is. When undo is required, the command must also save the state before execution and implement `undo()`. Multiple commands can be packaged into a macro command using the composite pattern.

1. Define the command interface, declaring `execute()` and optionally `undo()`.
2. A concrete command holds a receiver and calls it in `execute()`.
3. The invoker holds commands and triggers execution.
4. The client creates commands and assembles them into the invoker.
5. Save historical state when undo is required.

## Use Cases

- Button, menu, and keyboard-shortcut actions.
- Task queues, asynchronous execution, delayed execution.
- Undo/redo, transaction scripts, macro recording.

## Pros

- Decouples the invoker from the receiver.
- Supports queuing, logging, undo, and composition.
- Command objects can be reused and parameterized.

## Cons

- Every operation needs a command class, increasing the number of classes.
- Undo logic is complex to implement.
- Adds an indirection layer in simple scenarios.

## Java Example

```java
// Command interface: declares execute and undo
interface Command {
    // Execute the command
    void execute();

    // Undo the command
    void undo();
}

// Receiver: light
class Light {
    // Turn the light on
    public void on() {
        System.out.println("Light is on");
    }

    // Turn the light off
    public void off() {
        System.out.println("Light is off");
    }
}

// Light-on command
class LightOnCommand implements Command {
    private final Light light;

    // Inject the light object
    public LightOnCommand(Light light) {
        this.light = light;
    }

    // Execute light-on
    @Override
    public void execute() {
        light.on();
    }

    // Undo light-on: turn the light off
    @Override
    public void undo() {
        light.off();
    }
}

// Light-off command
class LightOffCommand implements Command {
    private final Light light;

    // Inject the light object
    public LightOffCommand(Light light) {
        this.light = light;
    }

    // Execute light-off
    @Override
    public void execute() {
        light.off();
    }

    // Undo light-off: turn the light on
    @Override
    public void undo() {
        light.on();
    }
}

// Invoker: the remote control holds only commands and does not care how they are executed
class RemoteControl {
    private Command command;

    // Set the command bound to the current button
    public void setCommand(Command command) {
        this.command = command;
    }

    // Press the button to execute the current command
    public void press() {
        command.execute();
    }

    // Press the undo button to undo the current command
    public void pressUndo() {
        command.undo();
    }
}

public class CommandDemo {
    // Entry point: assemble commands into the remote control and execute them
    public static void main(String[] args) {
        Light light = new Light();
        RemoteControl remote = new RemoteControl();

        remote.setCommand(new LightOnCommand(light));
        remote.press();
        remote.pressUndo();

        remote.setCommand(new LightOffCommand(light));
        remote.press();
    }
}
```

---

[Back to skill overview](../SKILL.md)
