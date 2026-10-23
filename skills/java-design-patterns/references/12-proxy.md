# 12. Proxy

> **Category**: Structural  
> **Intent**: Provide a surrogate or placeholder for another object to control access to it.

## Usage

The proxy and the real object implement the same interface, and the client accesses the real object through the proxy. The proxy can add control logic such as permissions, logging, transactions, and caching before and after calls, and can also delay creating the real object. Java commonly uses JDK dynamic proxies (interface-based) and CGLIB (subclass-based); Spring AOP chooses according to the target type.

1. Define the subject interface.
2. Implement the real subject.
3. Create a proxy that implements the same interface and holds the real object.
4. Add control logic in the proxy method, then call the real object.
5. The client uses the proxy through the interface.

## Use Cases

- Permission checks, logging, transactions, caching, rate limiting.
- Remote-call proxies, lazy loading, protecting the real object.
- Enhancing functionality without modifying the real object.

## Pros

- Controls access to the real object.
- Separates enhancement logic from business logic.
- Decouples the client from the real implementation.

## Cons

- Adds a layer of indirect calls.
- Dynamic proxies are harder to debug and troubleshoot.
- Overly deep proxy chains hurt performance.

## Java Example

```java
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;

// Subject interface: implemented by both the client-facing type and the real object
interface UserService {
    // Save a user
    void save(String name);
}

// Real subject: performs the actual business logic
class UserServiceImpl implements UserService {
    // Output the result of saving the user
    @Override
    public void save(String name) {
        System.out.println("Saving user: " + name);
    }
}

// Invocation handler: add logging before and after the real method
class LogHandler implements InvocationHandler {
    private final Object target;

    // Inject the real object
    public LogHandler(Object target) {
        this.target = target;
    }

    // Proxy entry point: log first, then invoke the real method reflectively
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("Invoking method: " + method.getName());
        Object result = method.invoke(target, args);
        System.out.println("Invocation finished");
        return result;
    }
}

public class ProxyDemo {
    // Entry point: generate a JDK dynamic proxy for UserService
    public static void main(String[] args) {
        UserService target = new UserServiceImpl();

        UserService proxy = (UserService) Proxy.newProxyInstance(
                target.getClass().getClassLoader(),
                target.getClass().getInterfaces(),
                new LogHandler(target)
        );

        proxy.save("Zhang San");
    }
}
```

---

[Back to skill overview](../SKILL.md)
