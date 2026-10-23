# 02. 工厂方法（Factory Method）

> **分类**：创建型  
> **意图**：定义创建对象的接口，让子类决定实例化哪个具体类。

## 使用说明

工厂方法把“创建哪个具体产品”的决定推迟到子类。客户端只依赖产品接口和工厂接口，新增产品时增加一个具体产品和一个具体工厂即可，不需要修改客户端。

1. 定义产品接口，声明产品需要提供的能力。
2. 编写具体产品类，实现产品接口。
3. 定义工厂接口，声明创建产品的方法。
4. 每个具体工厂返回对应的具体产品。
5. 客户端通过工厂获取产品，并面向产品接口调用。

## 使用场景

- 支付渠道、消息渠道、文件解析器等需要按类型创建对象的场景。
- 框架提供扩展点，由子类决定实例化哪个实现。
- 想把 `new` 集中管理，降低调用方与具体类的耦合。

## 优点

- 调用方与具体产品解耦。
- 新增产品符合开闭原则。
- 创建逻辑集中，便于统一处理初始化。

## 缺点

- 每个产品都要配一个工厂，类数量增加。
- 简单对象使用工厂会显得绕。
- 产品族变化时仍可能需要修改工厂接口。

## Java 示例

```java
// 产品接口：所有支付方式都必须实现支付方法
interface Payment {
    // 支付指定金额，单位为分
    void pay(long amountCents);
}

// 支付宝支付实现
class AlipayPayment implements Payment {
    // 执行支付宝支付
    @Override
    public void pay(long amountCents) {
        System.out.printf("支付宝支付 %.2f 元%n", amountCents / 100.0);
    }
}

// 微信支付实现
class WechatPayment implements Payment {
    // 执行微信支付
    @Override
    public void pay(long amountCents) {
        System.out.printf("微信支付 %.2f 元%n", amountCents / 100.0);
    }
}

// 工厂接口：声明创建支付产品的方法
interface PaymentFactory {
    // 创建一个支付产品
    Payment create();
}

// 支付宝工厂
class AlipayFactory implements PaymentFactory {
    // 返回支付宝支付实例
    @Override
    public Payment create() {
        return new AlipayPayment();
    }
}

// 微信工厂
class WechatFactory implements PaymentFactory {
    // 返回微信支付实例
    @Override
    public Payment create() {
        return new WechatPayment();
    }
}

public class FactoryMethodDemo {
    // 程序入口：通过工厂获取产品并调用
    public static void main(String[] args) {
        PaymentFactory factory = new WechatFactory();
        Payment payment = factory.create();
        payment.pay(1990);
    }
}
```

---

[返回技能说明](../SKILL.md)
