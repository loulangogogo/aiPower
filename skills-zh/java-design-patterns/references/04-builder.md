# 04. 建造者（Builder）

> **分类**：创建型  
> **意图**：将复杂对象的构建过程与表示分离，使同样的构建过程可以创建不同的表示。

## 使用说明

建造者把复杂对象的构造过程拆成若干步骤，最后统一调用 `build()`。常见写法是静态内部类 Builder：必填参数放构造方法，可选参数用链式方法设置，`build()` 里做校验并返回不可变对象。

1. 定义产品类，字段尽量设为 `final`。
2. 在内部定义 Builder，收集必填和可选参数。
3. 每个设置方法返回 `this`，支持链式调用。
4. 在 `build()` 中校验参数并创建产品。
5. 客户端按需组合参数，调用 `build()` 得到最终对象。

## 使用场景

- 参数多、可选参数多、容易传错位置的复杂对象。
- HTTP 请求、订单、SQL 查询、配置对象。
- 需要分步构建并保持对象不可变。

## 优点

- 构建过程可读，参数含义清晰。
- 支持不可变对象。
- 校验集中，避免产生半成品对象。
- 同一构建过程可以产出不同表示。

## 缺点

- 代码量增加，需要额外的 Builder。
- 简单对象使用会显得繁琐。
- 与 Lombok 等工具生成的 Builder 可能重复。

## Java 示例

```java
// 产品类：字段全部不可变，只能通过 Builder 创建
final class Order {
    private final String orderId;
    private final String product;
    private final int quantity;
    private final String coupon;
    private final boolean giftWrap;

    // 私有构造方法：只允许 Builder 调用
    private Order(Builder builder) {
        this.orderId = builder.orderId;
        this.product = builder.product;
        this.quantity = builder.quantity;
        this.coupon = builder.coupon;
        this.giftWrap = builder.giftWrap;
    }

    // 返回订单的文本表示，方便演示输出
    @Override
    public String toString() {
        return "订单 " + orderId
                + "：商品=" + product
                + "，数量=" + quantity
                + "，优惠券=" + (coupon == null ? "无" : coupon)
                + "，礼品包装=" + (giftWrap ? "是" : "否");
    }

    // 静态内部类 Builder：分步收集参数并校验
    public static class Builder {
        private final String orderId;
        private final String product;
        private int quantity = 1;
        private String coupon;
        private boolean giftWrap;

        // 创建 Builder，接收两个必填参数
        public Builder(String orderId, String product) {
            this.orderId = orderId;
            this.product = product;
        }

        // 设置购买数量
        public Builder quantity(int quantity) {
            this.quantity = quantity;
            return this;
        }

        // 设置优惠券，可选
        public Builder coupon(String coupon) {
            this.coupon = coupon;
            return this;
        }

        // 设置是否使用礼品包装
        public Builder giftWrap(boolean giftWrap) {
            this.giftWrap = giftWrap;
            return this;
        }

        // 校验参数并创建不可变的订单对象
        public Order build() {
            if (orderId == null || orderId.isBlank()) {
                throw new IllegalArgumentException("订单号不能为空");
            }
            if (product == null || product.isBlank()) {
                throw new IllegalArgumentException("商品名称不能为空");
            }
            if (quantity <= 0) {
                throw new IllegalArgumentException("数量必须大于 0");
            }
            return new Order(this);
        }
    }
}

public class BuilderDemo {
    // 程序入口：使用链式 Builder 组装订单
    public static void main(String[] args) {
        Order order = new Order.Builder("A001", "机械键盘")
                .quantity(2)
                .coupon("NEW10")
                .giftWrap(true)
                .build();

        System.out.println(order);
    }
}
```

---

[返回技能说明](../SKILL.md)
