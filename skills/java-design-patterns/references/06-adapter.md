# 06. Adapter

> **Category**: Structural  
> **Intent**: Convert the interface of a class into another interface the client expects, so that classes that would otherwise be incompatible can work together.

## Usage

The adapter converts the interface of an existing class into the interface the client expects. An object adapter holds the adaptee through composition and performs call conversion in the target method; a class adapter implements the conversion through inheritance, but is rarely used because Java only supports single inheritance. The conversion layer should only adapt interfaces and data; do not stuff business logic into it.

1. Define the target interface used by the client.
2. Keep the existing implementation and do not modify its code.
3. Create an adapter that implements the target interface and holds the adaptee.
4. Call the adaptee in the target method, converting parameters or return values.
5. The client depends only on the target interface.

## Use Cases

- Integrating a third-party SDK or a legacy system interface.
- Unifying the invocation style of multiple payment, SMS, or storage services.
- Upgrading an interface without changing a large number of callers.

## Pros

- Reuses existing implementations with low modification cost.
- Decouples the client from concrete implementations.
- Adapters can be tested independently.

## Cons

- Adds a layer of indirect calls.
- Too many adapters make the system structure more complex.
- When interface semantics differ, conversion may lose information.

## Java Example

```java
// SMS interface provided by the legacy system, whose method signature differs from the new system
class LegacySmsSender {
    // Legacy interface: send an SMS by phone number and content
    public void sendText(String phone, String content) {
        System.out.println("Legacy interface sending SMS to " + phone + ": " + content);
    }
}

// Target interface used uniformly by the new system
interface SmsSender {
    // Send an SMS
    void send(String phone, String content);
}

// Object adapter: compose the legacy interface and convert it to the new interface
class LegacySmsAdapter implements SmsSender {
    private final LegacySmsSender legacySender;

    // Inject the adaptee instance
    public LegacySmsAdapter(LegacySmsSender legacySender) {
        this.legacySender = legacySender;
    }

    // Call the legacy interface in the target method, converting parameters here if necessary
    @Override
    public void send(String phone, String content) {
        legacySender.sendText(phone, content);
    }
}

public class AdapterDemo {
    // Entry point: the client depends only on SmsSender
    public static void main(String[] args) {
        SmsSender sender = new LegacySmsAdapter(new LegacySmsSender());
        sender.send("13800000000", "Your verification code is 123456");
    }
}
```

---

[Back to skill overview](../SKILL.md)
