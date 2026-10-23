# 21. 策略（Strategy）

> **分类**：行为型  
> **意图**：定义一系列算法，把它们封装起来并使它们可以互相替换，让算法独立于使用它的客户而变化。

## 使用说明

策略模式把一组可互换的算法封装成独立的类。上下文持有策略接口，把计算委托给当前策略；客户端在运行时选择或注入具体策略。与状态模式不同，策略通常由外部选择，策略之间不自动转换。

1. 定义策略接口。
2. 每个算法实现策略接口。
3. 上下文持有策略并委托计算。
4. 客户端选择策略，或通过工厂/配置注入。
5. 处理未设置策略等边界情况。

## 使用场景

- 支付方式、折扣/优惠计算、运费计算。
- 排序、压缩、加密、限流算法。
- 需要运行时切换算法。

## 优点

- 算法可以独立替换。
- 避免大量 if/else。
- 符合开闭原则，便于测试。

## 缺点

- 客户端需要了解不同策略。
- 类数量增加。
- 策略之间难以共享中间状态。

## Java 示例

```java
// 策略接口：封装一种折扣算法
interface DiscountStrategy {
    // 根据原价计算折后价
    double apply(double originalPrice);
}

// 无折扣策略
class NoDiscount implements DiscountStrategy {
    // 原价返回
    @Override
    public double apply(double originalPrice) {
        return originalPrice;
    }
}

// 百分比折扣策略
class PercentageDiscount implements DiscountStrategy {
    private final double rate;

    // 保存折扣比例，例如 0.15 表示减免 15%
    public PercentageDiscount(double rate) {
        this.rate = rate;
    }

    // 按比例计算折后价
    @Override
    public double apply(double originalPrice) {
        return originalPrice * (1 - rate);
    }
}

// 满减策略
class FullReduction implements DiscountStrategy {
    private final double threshold;
    private final double reduction;

    // 保存满减门槛和减免金额
    public FullReduction(double threshold, double reduction) {
        this.threshold = threshold;
        this.reduction = reduction;
    }

    // 达到门槛时减固定金额
    @Override
    public double apply(double originalPrice) {
        return originalPrice >= threshold ? originalPrice - reduction : originalPrice;
    }
}

// 上下文：持有策略并委托计算
class PriceCalculator {
    private DiscountStrategy strategy;

    // 运行时切换折扣策略
    public void setStrategy(DiscountStrategy strategy) {
        this.strategy = strategy;
    }

    // 使用当前策略计算价格
    public double calculate(double originalPrice) {
        if (strategy == null) {
            throw new IllegalStateException("未设置折扣策略");
        }
        return strategy.apply(originalPrice);
    }
}

public class StrategyDemo {
    // 程序入口：同一计算器切换不同折扣策略
    public static void main(String[] args) {
        PriceCalculator calculator = new PriceCalculator();
        double price = 200.0;

        calculator.setStrategy(new NoDiscount());
        System.out.printf("原价：%.2f 元%n", calculator.calculate(price));

        calculator.setStrategy(new PercentageDiscount(0.15));
        System.out.printf("八五折：%.2f 元%n", calculator.calculate(price));

        calculator.setStrategy(new FullReduction(200, 30));
        System.out.printf("满 200 减 30：%.2f 元%n", calculator.calculate(price));
    }
}
```

---

[返回技能说明](../SKILL.md)
