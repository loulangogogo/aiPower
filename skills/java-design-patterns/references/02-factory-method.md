# 02. Factory Method

> **Category**: Creational  
> **Intent**: Define an interface for creating objects and let subclasses decide which concrete class to instantiate.

## Usage

The factory method defers the decision of "which concrete product to create" to subclasses. The client depends only on the product interface and the factory interface; adding a product requires only a new concrete product and a new concrete factory, without modifying the client.

1. Define the product interface and declare the capabilities a product must provide.
2. Write concrete product classes that implement the product interface.
3. Define the factory interface and declare the method that creates products.
4. Each concrete factory returns its corresponding concrete product.
5. The client obtains products through the factory and calls them against the product interface.

## Use Cases

- Scenarios that create objects by type, such as payment channels, messaging channels, and file parsers.
- A framework provides an extension point, and subclasses decide which implementation to instantiate.
- Centralize `new` to reduce coupling between callers and concrete classes.

## Pros

- Decouples callers from concrete products.
- Adding products follows the open-closed principle.
- Creation logic is centralized, making initialization easier to handle uniformly.

## Cons

- Every product needs a factory, increasing the number of classes.
- Using a factory for simple objects feels indirect.
- The factory interface may still need changes when the product family changes.

## Java Example

```java
// Product interface: every payment method must implement the payment method
interface Payment {
    // Pay the specified amount in cents
    void pay(long amountCents);
}

// Alipay payment implementation
class AlipayPayment implements Payment {
    // Execute Alipay payment
    @Override
    public void pay(long amountCents) {
        System.out.printf("Alipay payment: %.2f CNY%n", amountCents / 100.0);
    }
}

// WeChat Pay implementation
class WechatPayment implements Payment {
    // Execute WeChat Pay payment
    @Override
    public void pay(long amountCents) {
        System.out.printf("WeChat Pay payment: %.2f CNY%n", amountCents / 100.0);
    }
}

// Factory interface: declare the method that creates payment products
interface PaymentFactory {
    // Create a payment product
    Payment create();
}

// Alipay factory
class AlipayFactory implements PaymentFactory {
    // Return an Alipay payment instance
    @Override
    public Payment create() {
        return new AlipayPayment();
    }
}

// WeChat factory
class WechatFactory implements PaymentFactory {
    // Return a WeChat Pay instance
    @Override
    public Payment create() {
        return new WechatPayment();
    }
}

public class FactoryMethodDemo {
    // Entry point: obtain a product through a factory and call it
    public static void main(String[] args) {
        PaymentFactory factory = new WechatFactory();
        Payment payment = factory.create();
        payment.pay(1990);
    }
}
```

---

[Back to skill overview](../SKILL.md)
