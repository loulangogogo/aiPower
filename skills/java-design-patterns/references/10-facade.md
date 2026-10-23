# 10. Facade

> **Category**: Structural  
> **Intent**: Provide a unified high-level interface to a set of interfaces in a subsystem, making the subsystem easier to use.

## Usage

The facade pattern provides a unified high-level entry point for a group of subsystems. The Facade holds or calls multiple subsystem objects and encapsulates a complex call sequence into one or a few coarse-grained methods. The client interacts only with the Facade and does not need to understand the subsystem's internal collaboration. The Facade does not prevent the client from accessing subsystems directly; it merely offers a simpler path.

1. Identify a group of subsystem operations commonly used by the client.
2. Create a Facade class that composes the required subsystems.
3. Provide a small number of high-level methods that call the subsystems in order.
4. The client depends only on the Facade.
5. Keep the Facade's responsibilities in check so it does not grow without bound.

## Use Cases

- Startup/shutdown flows: CPU, memory, disk.
- Order placement flows: inventory, payment, points, notification.
- Provide a unified entry point within a project for a complex SDK.

## Pros

- Reduces the client's cost of use.
- Reduces coupling between the client and the subsystems.
- Makes the system layering clearer.

## Cons

- The Facade can easily grow into a god class.
- Excessive encapsulation hides necessary details.
- It cannot prevent clients from bypassing the Facade.

## Java Example

```java
// Subsystem: CPU
class Cpu {
    // Start the CPU
    public void start() {
        System.out.println("CPU started");
    }

    // Shut down the CPU
    public void shutdown() {
        System.out.println("CPU shut down");
    }
}

// Subsystem: memory
class Memory {
    // Load memory
    public void load() {
        System.out.println("Memory loaded");
    }

    // Release memory
    public void release() {
        System.out.println("Memory released");
    }
}

// Subsystem: disk
class Disk {
    // Read the disk
    public void read() {
        System.out.println("Disk read");
    }

    // Stop the disk
    public void stop() {
        System.out.println("Disk stopped");
    }
}

// Facade: orchestrate subsystem calls into a simple entry point
class ComputerFacade {
    private final Cpu cpu = new Cpu();
    private final Memory memory = new Memory();
    private final Disk disk = new Disk();

    // Start each subsystem in order
    public void start() {
        cpu.start();
        memory.load();
        disk.read();
        System.out.println("Computer started");
    }

    // Shut down each subsystem in reverse order
    public void shutdown() {
        disk.stop();
        memory.release();
        cpu.shutdown();
        System.out.println("Computer shut down");
    }
}

public class FacadeDemo {
    // Entry point: the client interacts only with the facade
    public static void main(String[] args) {
        ComputerFacade computer = new ComputerFacade();
        computer.start();
        computer.shutdown();
    }
}
```

---

[Back to skill overview](../SKILL.md)
