# 20. State

> **Category**: Behavioral  
> **Intent**: Allow an object to alter its behavior when its internal state changes, so that the object appears to change its class.

## Usage

The state pattern encapsulates an object's behavior in different states into separate classes. The context holds the current state and delegates requests to the state object; the state object triggers transitions at the appropriate time. Unlike the strategy pattern, state transitions are usually driven automatically by internal events, and the client generally does not need to select a state directly.

1. Define the state interface and declare the operations supported in that state.
2. Each concrete state implements the behavior.
3. The context holds the current state and delegates requests.
4. Perform state transitions in the state object or the context.
5. Map out the state-transition diagram centrally to avoid omissions.

## Use Cases

- Order states: pending payment, paid, shipped, cancelled.
- Elevators, traffic lights, players, workflow engines.
- Large amounts of state-related if/switch logic that must be eliminated.

## Pros

- Eliminates conditional branches that test state.
- Each state has a single responsibility.
- State transitions are clear, making new states easy to add.

## Cons

- The number of state classes increases.
- When transitions are scattered, it is hard to grasp the whole picture.
- Using it for a simple state machine is over-engineering.

## Java Example

```java
// State interface: declares the operations an order supports in each state
interface OrderState {
    // Pay for the order
    void pay(OrderContext order);

    // Ship the order
    void ship(OrderContext order);

    // Cancel the order
    void cancel(OrderContext order);
}

// Pending-payment state
class PendingPaymentState implements OrderState {
    // Switch to the paid state after successful payment
    @Override
    public void pay(OrderContext order) {
        System.out.println("Payment successful");
        order.setState(new PaidState());
    }

    // Shipping is forbidden while unpaid
    @Override
    public void ship(OrderContext order) {
        System.out.println("Not paid; cannot ship");
    }

    // Cancel the order and switch to the cancelled state
    @Override
    public void cancel(OrderContext order) {
        System.out.println("Order cancelled");
        order.setState(new CancelledState());
    }
}

// Paid state
class PaidState implements OrderState {
    // Repeated payment is forbidden once paid
    @Override
    public void pay(OrderContext order) {
        System.out.println("Order already paid; do not pay again");
    }

    // Switch to the shipped state after shipping
    @Override
    public void ship(OrderContext order) {
        System.out.println("Order shipped");
        order.setState(new ShippedState());
    }

    // A paid order cannot be cancelled directly
    @Override
    public void cancel(OrderContext order) {
        System.out.println("Paid orders must go through the refund process");
    }
}

// Shipped state
class ShippedState implements OrderState {
    // Payment is forbidden once shipped
    @Override
    public void pay(OrderContext order) {
        System.out.println("Order shipped; cannot pay");
    }

    // Repeated shipping is forbidden once shipped
    @Override
    public void ship(OrderContext order) {
        System.out.println("Order shipped; do not ship again");
    }

    // A shipped order cannot be cancelled directly
    @Override
    public void cancel(OrderContext order) {
        System.out.println("Shipped orders cannot be cancelled directly");
    }
}

// Cancelled state
class CancelledState implements OrderState {
    // Payment is forbidden once cancelled
    @Override
    public void pay(OrderContext order) {
        System.out.println("Order cancelled; cannot pay");
    }

    // Shipping is forbidden once cancelled
    @Override
    public void ship(OrderContext order) {
        System.out.println("Order cancelled; cannot ship");
    }

    // Report when cancel is requested again
    @Override
    public void cancel(OrderContext order) {
        System.out.println("Order cancelled");
    }
}

// Context: holds the current state and delegates requests to the state object
class OrderContext {
    private OrderState state = new PendingPaymentState();

    // Switch the current state
    public void setState(OrderState state) {
        this.state = state;
    }

    // Initiate a payment request
    public void pay() {
        state.pay(this);
    }

    // Initiate a shipping request
    public void ship() {
        state.ship(this);
    }

    // Initiate a cancellation request
    public void cancel() {
        state.cancel(this);
    }
}

public class StateDemo {
    // Entry point: pay, ship, and cancel in order, observing state changes
    public static void main(String[] args) {
        OrderContext order = new OrderContext();
        order.pay();
        order.ship();
        order.cancel();
    }
}
```

---

[Back to skill overview](../SKILL.md)
