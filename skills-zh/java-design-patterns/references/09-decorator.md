# 09. 装饰器（Decorator）

> **分类**：结构型  
> **意图**：动态地给一个对象添加一些额外的职责。就增加功能而言，装饰器比生成子类更灵活。

## 使用说明

装饰器在保持接口不变的前提下，通过包装对象来增加功能。装饰器实现与组件相同的接口，并持有一个组件实例；调用时先执行自己的增强逻辑，再委托给被包装对象。多个装饰器可以层层叠加，顺序会影响最终结果。

1. 定义组件接口。
2. 实现基础组件。
3. 定义装饰器基类，实现组件接口并持有组件。
4. 每个具体装饰器在委托前后添加行为。
5. 客户端按需叠加装饰器。

## 使用场景

- Java IO 流：缓冲、压缩、加密。
- 给接口增加日志、缓存、限流、重试、权限。
- 需要在运行时动态组合功能。

## 优点

- 比继承灵活，功能可以动态叠加。
- 每个装饰器职责单一。
- 避免为每种组合生成子类。

## 缺点

- 会产生大量小对象。
- 装饰顺序敏感，排查问题较难。
- 多层包装会增加调用开销。

## Java 示例

```java
// 组件接口：咖啡的描述和价格
interface Coffee {
    // 返回咖啡描述
    String description();

    // 返回咖啡价格
    double cost();
}

// 基础组件：美式咖啡
class SimpleCoffee implements Coffee {
    // 返回基础描述
    @Override
    public String description() {
        return "美式咖啡";
    }

    // 返回基础价格
    @Override
    public double cost() {
        return 18.0;
    }
}

// 装饰器基类：持有被装饰的咖啡
abstract class CoffeeDecorator implements Coffee {
    protected final Coffee coffee;

    // 注入需要装饰的咖啡对象
    protected CoffeeDecorator(Coffee coffee) {
        this.coffee = coffee;
    }
}

// 加牛奶装饰器
class MilkDecorator extends CoffeeDecorator {
    // 创建牛奶装饰器
    public MilkDecorator(Coffee coffee) {
        super(coffee);
    }

    // 在原有描述后追加牛奶
    @Override
    public String description() {
        return coffee.description() + " + 牛奶";
    }

    // 在原有价格上增加 3 元
    @Override
    public double cost() {
        return coffee.cost() + 3.0;
    }
}

// 加糖装饰器
class SugarDecorator extends CoffeeDecorator {
    // 创建糖装饰器
    public SugarDecorator(Coffee coffee) {
        super(coffee);
    }

    // 在原有描述后追加糖
    @Override
    public String description() {
        return coffee.description() + " + 糖";
    }

    // 在原有价格上增加 1 元
    @Override
    public double cost() {
        return coffee.cost() + 1.0;
    }
}

public class DecoratorDemo {
    // 程序入口：叠加牛奶和糖装饰器
    public static void main(String[] args) {
        Coffee coffee = new SugarDecorator(new MilkDecorator(new SimpleCoffee()));

        System.out.println(coffee.description());
        System.out.printf("总价：%.2f 元%n", coffee.cost());
    }
}
```

---

[返回技能说明](../SKILL.md)
