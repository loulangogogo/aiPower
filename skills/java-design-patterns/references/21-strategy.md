# 21. Strategy

> **Category**: Behavioral  
> **Intent**: Define a family of algorithms, encapsulate each one, and make them interchangeable, letting the algorithm vary independently from the clients that use it.

## Usage

The strategy pattern encapsulates a set of interchangeable algorithms into separate classes. The context holds the strategy interface and delegates computation to the current strategy; the client selects or injects a concrete strategy at runtime. Unlike the state pattern, a strategy is usually selected externally, and strategies do not switch automatically.

1. Define the strategy interface.
2. Each algorithm implements the strategy interface.
3. The context holds a strategy and delegates computation.
4. The client selects a strategy, or injects one through a factory/configuration.
5. Handle edge cases such as no strategy being set.

## Use Cases

- Payment methods, discount/promotion calculations, shipping-fee calculations.
- Sorting, compression, encryption, and rate-limiting algorithms.
- Switching algorithms at runtime.

## Pros

- Algorithms can be replaced independently.
- Avoids large if/else blocks.
- Follows the open-closed principle and is easy to test.

## Cons

- The client needs to know the different strategies.
- The number of classes increases.
- Strategies have difficulty sharing intermediate state.

## Java Example

```java
// Strategy interface: encapsulates a discount algorithm
interface DiscountStrategy {
    // Compute the discounted price from the original price
    double apply(double originalPrice);
}

// No-discount strategy
class NoDiscount implements DiscountStrategy {
    // Return the original price
    @Override
    public double apply(double originalPrice) {
        return originalPrice;
    }
}

// Percentage-discount strategy
class PercentageDiscount implements DiscountStrategy {
    private final double rate;

    // Store the discount rate, for example 0.15 means 15% off
    public PercentageDiscount(double rate) {
        this.rate = rate;
    }

    // Compute the discounted price by rate
    @Override
    public double apply(double originalPrice) {
        return originalPrice * (1 - rate);
    }
}

// Threshold-reduction strategy
class FullReduction implements DiscountStrategy {
    private final double threshold;
    private final double reduction;

    // Store the threshold and the reduction amount
    public FullReduction(double threshold, double reduction) {
        this.threshold = threshold;
        this.reduction = reduction;
    }

    // Subtract a fixed amount when the threshold is reached
    @Override
    public double apply(double originalPrice) {
        return originalPrice >= threshold ? originalPrice - reduction : originalPrice;
    }
}

// Context: holds a strategy and delegates computation
class PriceCalculator {
    private DiscountStrategy strategy;

    // Switch the discount strategy at runtime
    public void setStrategy(DiscountStrategy strategy) {
        this.strategy = strategy;
    }

    // Compute the price with the current strategy
    public double calculate(double originalPrice) {
        if (strategy == null) {
            throw new IllegalStateException("Discount strategy not set");
        }
        return strategy.apply(originalPrice);
    }
}

public class StrategyDemo {
    // Entry point: switch different discount strategies on the same calculator
    public static void main(String[] args) {
        PriceCalculator calculator = new PriceCalculator();
        double price = 200.0;

        calculator.setStrategy(new NoDiscount());
        System.out.printf("Original price: %.2f CNY%n", calculator.calculate(price));

        calculator.setStrategy(new PercentageDiscount(0.15));
        System.out.printf("15%% off: %.2f CNY%n", calculator.calculate(price));

        calculator.setStrategy(new FullReduction(200, 30));
        System.out.printf("Spend 200, save 30: %.2f CNY%n", calculator.calculate(price));
    }
}
```

---

[Back to skill overview](../SKILL.md)
