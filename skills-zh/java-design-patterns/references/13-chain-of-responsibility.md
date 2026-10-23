# 13. 责任链（Chain of Responsibility）

> **分类**：行为型  
> **意图**：使多个对象都有机会处理请求，从而避免请求的发送者和接收者之间的耦合。

## 使用说明

责任链把多个处理者连成一条链，每个处理者判断自己能否处理请求：能处理就处理，不能处理就转给下一个。链的顺序决定优先级，客户端只把请求交给链头。需要明确无人处理时的兜底逻辑，并避免链中形成环。

1. 定义处理者接口，包含处理方法和下一个处理者的引用。
2. 每个具体处理者实现自己的判断和处理逻辑。
3. 无法处理时调用下一个处理者。
4. 客户端组装链并交给链头。
5. 设置兜底处理或明确失败行为。

## 使用场景

- 审批流、参数校验、过滤器/拦截器。
- 日志分级、限流规则、风控规则。
- Servlet Filter、Spring Security、Netty Pipeline。

## 优点

- 请求发送者与处理者解耦。
- 链可以动态调整和复用。
- 每个处理者职责单一。

## 缺点

- 请求可能无人处理，需要兜底。
- 链过长会影响性能。
- 调用路径不直观，调试困难。

## Java 示例

```java
// 抽象处理者：持有下一个处理者，并声明审批方法
abstract class Approver {
    protected Approver next;

    // 设置下一个处理者，并返回它以便链式组装
    public Approver setNext(Approver next) {
        this.next = next;
        return next;
    }

    // 处理请假申请，由具体处理者实现
    public abstract void approve(int days);
}

// 组长：最多批准 1 天
class TeamLeader extends Approver {
    // 处理 1 天以内的请假，否则转给下一个
    @Override
    public void approve(int days) {
        if (days <= 1) {
            System.out.println("组长批准 " + days + " 天请假");
        } else if (next != null) {
            next.approve(days);
        }
    }
}

// 经理：最多批准 3 天
class Manager extends Approver {
    // 处理 3 天以内的请假，否则转给下一个
    @Override
    public void approve(int days) {
        if (days <= 3) {
            System.out.println("经理批准 " + days + " 天请假");
        } else if (next != null) {
            next.approve(days);
        }
    }
}

// 总监：最多批准 7 天，超出权限时给出兜底结果
class Director extends Approver {
    // 处理 7 天以内的请假，否则提示超出权限
    @Override
    public void approve(int days) {
        if (days <= 7) {
            System.out.println("总监批准 " + days + " 天请假");
        } else {
            System.out.println(days + " 天请假超出审批权限");
        }
    }
}

public class ChainDemo {
    // 程序入口：组装审批链并提交不同天数的申请
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

[返回技能说明](../SKILL.md)
