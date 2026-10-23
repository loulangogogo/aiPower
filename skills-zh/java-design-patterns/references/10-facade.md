# 10. 外观（Facade）

> **分类**：结构型  
> **意图**：为子系统中的一组接口提供一个统一的高层接口，使子系统更容易使用。

## 使用说明

外观模式为一组子系统提供一个统一的高层入口。Facade 持有或调用多个子系统对象，把复杂的调用顺序封装成一个或几个粗粒度方法。客户端只和 Facade 交互，不需要了解子系统的内部协作。Facade 不会阻止客户端直接访问子系统，它只是提供一条更简单的路径。

1. 识别客户端常用的一组子系统操作。
2. 创建 Facade 类，组合所需子系统。
3. 提供少量高层方法，内部按顺序调用子系统。
4. 客户端只依赖 Facade。
5. 控制 Facade 的职责范围，避免它无限膨胀。

## 使用场景

- 启动/关闭流程：CPU、内存、磁盘。
- 下单流程：库存、支付、积分、通知。
- 为复杂 SDK 提供项目内统一入口。

## 优点

- 降低客户端的使用成本。
- 减少客户端与子系统的耦合。
- 让系统层次更清晰。

## 缺点

- Facade 容易膨胀成上帝类。
- 过度封装会隐藏必要的细节。
- 不能阻止客户端绕过 Facade。

## Java 示例

```java
// 子系统：CPU
class Cpu {
    // 启动 CPU
    public void start() {
        System.out.println("CPU 启动");
    }

    // 关闭 CPU
    public void shutdown() {
        System.out.println("CPU 关闭");
    }
}

// 子系统：内存
class Memory {
    // 加载内存
    public void load() {
        System.out.println("内存加载");
    }

    // 释放内存
    public void release() {
        System.out.println("内存释放");
    }
}

// 子系统：磁盘
class Disk {
    // 读取磁盘
    public void read() {
        System.out.println("磁盘读取");
    }

    // 停止磁盘
    public void stop() {
        System.out.println("磁盘停止");
    }
}

// 外观：把多个子系统的调用编排成简单入口
class ComputerFacade {
    private final Cpu cpu = new Cpu();
    private final Memory memory = new Memory();
    private final Disk disk = new Disk();

    // 按顺序启动各个子系统
    public void start() {
        cpu.start();
        memory.load();
        disk.read();
        System.out.println("计算机启动完成");
    }

    // 按相反顺序关闭各个子系统
    public void shutdown() {
        disk.stop();
        memory.release();
        cpu.shutdown();
        System.out.println("计算机已关闭");
    }
}

public class FacadeDemo {
    // 程序入口：客户端只与外观交互
    public static void main(String[] args) {
        ComputerFacade computer = new ComputerFacade();
        computer.start();
        computer.shutdown();
    }
}
```

---

[返回技能说明](../SKILL.md)
