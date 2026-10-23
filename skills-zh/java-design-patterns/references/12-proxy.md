# 12. 代理（Proxy）

> **分类**：结构型  
> **意图**：为其他对象提供一种代理以控制对这个对象的访问。

## 使用说明

代理和真实对象实现同一个接口，客户端通过代理访问真实对象。代理可以在调用前后加入权限、日志、事务、缓存等控制逻辑，也可以延迟创建真实对象。Java 常用 JDK 动态代理（基于接口）和 CGLIB（基于子类），Spring AOP 会根据目标类型选择。

1. 定义主题接口。
2. 实现真实主题。
3. 创建代理，实现同一接口并持有真实对象。
4. 在代理方法中加入控制逻辑，再调用真实对象。
5. 客户端面向接口使用代理。

## 使用场景

- 权限校验、日志、事务、缓存、限流。
- 远程调用代理、懒加载、保护真实对象。
- 需要在不修改真实对象的情况下增强功能。

## 优点

- 控制对真实对象的访问。
- 增强逻辑与业务逻辑分离。
- 客户端与真实实现解耦。

## 缺点

- 增加一层间接调用。
- 动态代理调试和排查更困难。
- 代理链过深会影响性能。

## Java 示例

```java
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;

// 主题接口：客户端和真实对象共同实现
interface UserService {
    // 保存用户
    void save(String name);
}

// 真实主题：真正执行业务逻辑
class UserServiceImpl implements UserService {
    // 输出保存用户的业务结果
    @Override
    public void save(String name) {
        System.out.println("保存用户：" + name);
    }
}

// 调用处理器：在真实方法前后加入日志
class LogHandler implements InvocationHandler {
    private final Object target;

    // 注入真实对象
    public LogHandler(Object target) {
        this.target = target;
    }

    // 代理调用入口：先记日志，再反射调用真实方法
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("调用方法：" + method.getName());
        Object result = method.invoke(target, args);
        System.out.println("调用结束");
        return result;
    }
}

public class ProxyDemo {
    // 程序入口：为 UserService 生成 JDK 动态代理
    public static void main(String[] args) {
        UserService target = new UserServiceImpl();

        UserService proxy = (UserService) Proxy.newProxyInstance(
                target.getClass().getClassLoader(),
                target.getClass().getInterfaces(),
                new LogHandler(target)
        );

        proxy.save("张三");
    }
}
```

---

[返回技能说明](../SKILL.md)
