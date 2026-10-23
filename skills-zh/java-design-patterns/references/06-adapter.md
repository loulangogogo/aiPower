# 06. 适配器（Adapter）

> **分类**：结构型  
> **意图**：将一个类的接口转换成客户端期望的另一个接口，使原本不兼容的类可以一起工作。

## 使用说明

适配器把已有类的接口转换成客户端期望的接口。对象适配器通过组合持有被适配对象，在目标方法里做调用转换；类适配器通过继承实现，但受 Java 单继承限制，较少使用。转换层只做接口和数据的适配，不要把业务逻辑塞进去。

1. 定义客户端使用的目标接口。
2. 保留现有实现，不修改它的代码。
3. 创建适配器，实现目标接口并持有被适配对象。
4. 在目标方法中调用被适配对象，完成参数或返回值转换。
5. 客户端只依赖目标接口。

## 使用场景

- 接入第三方 SDK 或旧系统接口。
- 统一多个支付、短信、存储服务的调用方式。
- 接口升级但不想改动大量调用方。

## 优点

- 复用已有实现，改造成本低。
- 客户端与具体实现解耦。
- 适配器可以独立测试。

## 缺点

- 增加一层间接调用。
- 适配器过多会让系统结构变复杂。
- 接口语义不一致时，转换可能丢失信息。

## Java 示例

```java
// 旧系统提供的短信接口，方法签名与新系统不同
class LegacySmsSender {
    // 旧接口：按手机号和内容发送短信
    public void sendText(String phone, String content) {
        System.out.println("旧接口发送短信到 " + phone + "：" + content);
    }
}

// 新系统统一使用的目标接口
interface SmsSender {
    // 发送短信
    void send(String phone, String content);
}

// 对象适配器：组合旧接口，转换为新接口
class LegacySmsAdapter implements SmsSender {
    private final LegacySmsSender legacySender;

    // 注入被适配的旧接口实例
    public LegacySmsAdapter(LegacySmsSender legacySender) {
        this.legacySender = legacySender;
    }

    // 在目标方法中调用旧接口，必要时在这里做参数转换
    @Override
    public void send(String phone, String content) {
        legacySender.sendText(phone, content);
    }
}

public class AdapterDemo {
    // 程序入口：客户端只依赖 SmsSender
    public static void main(String[] args) {
        SmsSender sender = new LegacySmsAdapter(new LegacySmsSender());
        sender.send("13800000000", "您的验证码是 123456");
    }
}
```

---

[返回技能说明](../SKILL.md)
