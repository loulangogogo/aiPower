# 07. 桥接（Bridge）

> **分类**：结构型  
> **意图**：将抽象部分与它的实现部分分离，使它们都可以独立地变化。

## 使用说明

桥接把两个独立变化的维度拆开：抽象层持有实现层接口，抽象层的方法委托给实现层。这样新增“抽象种类”或“实现方式”都只需要扩展各自的分支，不会产生 M×N 个子类。它要求在设计初期识别出这两个维度。

1. 找出系统中独立变化的两个维度。
2. 定义实现维度的接口。
3. 在抽象维度中持有实现接口，构造时注入。
4. 抽象方法委托给实现对象。
5. 两个维度分别扩展子类。

## 使用场景

- 消息类型 × 发送渠道：普通/紧急 × 短信/邮件/站内信。
- 图形形状 × 渲染方式。
- 跨平台应用 × 平台能力实现。

## 优点

- 两个维度独立扩展，避免类爆炸。
- 用组合替代继承，结构更灵活。
- 抽象层和实现层可以分别演进。

## 缺点

- 抽象程度提高，理解和设计成本增加。
- 需要提前识别变化维度，识别错误会适得其反。
- 类比继承方案更多。

## Java 示例

```java
// 实现维度：发送渠道
interface MessageSender {
    // 通过当前渠道发送内容
    void send(String content);
}

// 短信渠道
class SmsSender implements MessageSender {
    // 输出短信发送结果
    @Override
    public void send(String content) {
        System.out.println("短信渠道：" + content);
    }
}

// 邮件渠道
class EmailSender implements MessageSender {
    // 输出邮件发送结果
    @Override
    public void send(String content) {
        System.out.println("邮件渠道：" + content);
    }
}

// 抽象维度：消息类型，持有发送渠道
abstract class Message {
    protected final MessageSender sender;

    // 注入具体发送渠道
    protected Message(MessageSender sender) {
        this.sender = sender;
    }

    // 发送消息，由子类决定消息格式
    public abstract void send(String content);
}

// 普通消息
class NormalMessage extends Message {
    // 创建普通消息并指定发送渠道
    public NormalMessage(MessageSender sender) {
        super(sender);
    }

    // 添加普通消息前缀后发送
    @Override
    public void send(String content) {
        sender.send("【普通】" + content);
    }
}

// 紧急消息
class UrgentMessage extends Message {
    // 创建紧急消息并指定发送渠道
    public UrgentMessage(MessageSender sender) {
        super(sender);
    }

    // 添加紧急前缀并触发额外提醒
    @Override
    public void send(String content) {
        sender.send("【紧急】" + content);
        System.out.println("同时触发电话提醒");
    }
}

public class BridgeDemo {
    // 程序入口：组合消息类型与发送渠道
    public static void main(String[] args) {
        Message normal = new NormalMessage(new EmailSender());
        Message urgent = new UrgentMessage(new SmsSender());

        normal.send("系统将于今晚维护");
        urgent.send("支付通道异常");
    }
}
```

---

[返回技能说明](../SKILL.md)
