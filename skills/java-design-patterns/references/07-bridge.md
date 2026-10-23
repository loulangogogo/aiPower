# 07. Bridge

> **Category**: Structural  
> **Intent**: Decouple an abstraction from its implementation so that the two can vary independently.

## Usage

The bridge separates two independently varying dimensions: the abstraction layer holds the implementation-layer interface, and methods of the abstraction delegate to the implementation layer. This way, adding an "abstraction kind" or an "implementation style" only requires extending its own branch, without producing M×N subclasses. It requires identifying these two dimensions early in design.

1. Find the two dimensions in the system that vary independently.
2. Define the interface of the implementation dimension.
3. Hold the implementation interface in the abstraction dimension and inject it through the constructor.
4. Delegate abstract methods to the implementation object.
5. Extend subclasses separately in the two dimensions.

## Use Cases

- Message type × sending channel: normal/urgent × SMS/email/in-app message.
- Graphic shape × rendering method.
- Cross-platform application × platform capability implementation.

## Pros

- The two dimensions extend independently, avoiding class explosion.
- Uses composition instead of inheritance, making the structure more flexible.
- The abstraction layer and implementation layer can evolve separately.

## Cons

- The abstraction level rises, increasing understanding and design costs.
- Change dimensions must be identified in advance; identifying them incorrectly backfires.
- More classes than an inheritance-based solution.

## Java Example

```java
// Implementation dimension: sending channel
interface MessageSender {
    // Send content through the current channel
    void send(String content);
}

// SMS channel
class SmsSender implements MessageSender {
    // Output the SMS sending result
    @Override
    public void send(String content) {
        System.out.println("SMS channel: " + content);
    }
}

// Email channel
class EmailSender implements MessageSender {
    // Output the email sending result
    @Override
    public void send(String content) {
        System.out.println("Email channel: " + content);
    }
}

// Abstraction dimension: message type, holding a sending channel
abstract class Message {
    protected final MessageSender sender;

    // Inject the concrete sending channel
    protected Message(MessageSender sender) {
        this.sender = sender;
    }

    // Send a message; subclasses decide the message format
    public abstract void send(String content);
}

// Normal message
class NormalMessage extends Message {
    // Create a normal message with the specified sending channel
    public NormalMessage(MessageSender sender) {
        super(sender);
    }

    // Send after adding the normal-message prefix
    @Override
    public void send(String content) {
        sender.send("[Normal] " + content);
    }
}

// Urgent message
class UrgentMessage extends Message {
    // Create an urgent message with the specified sending channel
    public UrgentMessage(MessageSender sender) {
        super(sender);
    }

    // Add the urgent prefix and trigger an extra alert
    @Override
    public void send(String content) {
        sender.send("[Urgent] " + content);
        System.out.println("Also triggering a phone alert");
    }
}

public class BridgeDemo {
    // Entry point: combine message types with sending channels
    public static void main(String[] args) {
        Message normal = new NormalMessage(new EmailSender());
        Message urgent = new UrgentMessage(new SmsSender());

        normal.send("The system will undergo maintenance tonight");
        urgent.send("Payment channel failure");
    }
}
```

---

[Back to skill overview](../SKILL.md)
