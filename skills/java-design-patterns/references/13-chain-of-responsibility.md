# 13. Chain of Responsibility

> **Category**: Behavioral  
> **Intent**: Avoid coupling the sender of a request to its receiver by giving multiple objects a chance to handle the request.

## Usage

The chain of responsibility links multiple handlers into a chain. Each handler decides whether it can handle the request: if it can, it handles it; otherwise it passes the request to the next handler. The order of the chain determines priority, and the client only hands the request to the head of the chain. Define explicit fallback behavior for when no one handles the request, and avoid cycles in the chain.

1. Define the handler interface, including a handling method and a reference to the next handler.
2. Each concrete handler implements its own decision and handling logic.
3. Call the next handler when it cannot handle the request.
4. The client assembles the chain and hands it the head.
5. Set up fallback handling or explicitly define the failure behavior.

## Use Cases

- Approval flows, parameter validation, filters/interceptors.
- Log leveling, rate-limiting rules, risk-control rules.
- Servlet Filter, Spring Security, Netty Pipeline.

## Pros

- Decouples request senders from handlers.
- The chain can be adjusted and reused dynamically.
- Each handler has a single responsibility.

## Cons

- A request may go unhandled, so a fallback is needed.
- A long chain hurts performance.
- The call path is not intuitive, making debugging difficult.

## Java Example

```java
// Abstract handler: holds the next handler and declares the approval method
abstract class Approver {
    protected Approver next;

    // Set the next handler and return it for chained assembly
    public Approver setNext(Approver next) {
        this.next = next;
        return next;
    }

    // Handle a leave request; implemented by concrete handlers
    public abstract void approve(int days);
}

// Team lead: approves up to 1 day
class TeamLeader extends Approver {
    // Handle leave within 1 day; otherwise pass it to the next handler
    @Override
    public void approve(int days) {
        if (days <= 1) {
            System.out.println("Team lead approved " + days + " day(s) of leave");
        } else if (next != null) {
            next.approve(days);
        }
    }
}

// Manager: approves up to 3 days
class Manager extends Approver {
    // Handle leave within 3 days; otherwise pass it to the next handler
    @Override
    public void approve(int days) {
        if (days <= 3) {
            System.out.println("Manager approved " + days + " day(s) of leave");
        } else if (next != null) {
            next.approve(days);
        }
    }
}

// Director: approves up to 7 days, and gives a fallback result when the request exceeds authority
class Director extends Approver {
    // Handle leave within 7 days; otherwise report that it exceeds authority
    @Override
    public void approve(int days) {
        if (days <= 7) {
            System.out.println("Director approved " + days + " day(s) of leave");
        } else {
            System.out.println(days + " day(s) of leave exceeds approval authority");
        }
    }
}

public class ChainDemo {
    // Entry point: assemble the approval chain and submit requests for different numbers of days
    public static void main(String[] args) {
        Approver leader = new TeamLeader();
        Approver manager = new Manager();
        Approver director = new Director();

        leader.setNext(manager).setNext(director);

        leader.approve(1);
        leader.approve(3);
        leader.approve(5);
        leader.approve(10);
    }
}
```

---

[Back to skill overview](../SKILL.md)
