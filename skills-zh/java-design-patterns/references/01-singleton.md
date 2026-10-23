# 01. 单例模式（Singleton）

> **分类**：创建型  
> **意图**：确保一个类只有一个实例，并提供一个全局访问点。

## 使用说明

单例类自己负责创建、保存并暴露唯一实例。实现时先私有化构造方法，再在类内部创建静态实例，最后通过静态方法返回它。懒加载可以用静态内部类；需要防止反射和反序列化破坏单例时，枚举是更稳妥的选择。

1. 私有化构造方法，禁止外部直接 `new`。
2. 在类内部创建并保存唯一实例。
3. 提供静态访问方法，统一返回该实例。
4. 根据场景选择饿汉、静态内部类或枚举，并处理并发与反射问题。

## 使用场景

- 全局配置、应用级缓存、日志器。
- 连接池、线程池等需要集中管理的资源。
- 需要严格控制实例数量的硬件或系统资源。

## 优点

- 控制实例数量，减少重复创建的开销。
- 提供统一访问入口，状态集中。
- 可以延迟初始化，按需创建。

## 缺点

- 全局状态让依赖关系变得隐式，单元测试难以替换。
- 反射、反序列化和多线程初始化都有陷阱。
- 滥用会掩盖模块边界问题。

## Java 示例

```java
public class SingletonDemo {
    // 静态内部类持有唯一实例，第一次访问时才初始化，天然线程安全
    private static class Holder {
        private static final SingletonDemo INSTANCE = new SingletonDemo();
    }

    // 私有构造方法，防止外部直接创建实例
    private SingletonDemo() {
        // 防止通过反射重复创建实例
        if (Holder.INSTANCE != null) {
            throw new IllegalStateException("单例已被初始化");
        }
    }

    // 返回全局唯一实例
    public static SingletonDemo getInstance() {
        return Holder.INSTANCE;
    }

    // 演示单例的业务方法
    public void log(String message) {
        System.out.println("[LOG] " + message);
    }

    // 程序入口：验证多次获取到的是同一个对象
    public static void main(String[] args) {
        SingletonDemo first = SingletonDemo.getInstance();
        SingletonDemo second = SingletonDemo.getInstance();

        System.out.println("是否为同一实例：" + (first == second));
        first.log("应用启动");
    }
}
```

---

[返回技能说明](../SKILL.md)
