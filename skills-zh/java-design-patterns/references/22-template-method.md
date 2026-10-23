# 22. 模板方法（Template Method）

> **分类**：行为型  
> **意图**：定义一个操作中的算法骨架，而将一些步骤延迟到子类中，使子类可以不改变算法结构就重定义某些步骤。

## 使用说明

模板方法在抽象类中定义算法骨架，把可变步骤留给子类。骨架方法通常声明为 `final`，防止子类破坏流程；可变步骤可以是抽象方法，也可以是带默认实现的钩子方法。它依赖继承，适合流程稳定、步骤差异明确的场景。

1. 在抽象类中定义 `final` 模板方法。
2. 把固定步骤实现为私有方法。
3. 把可变步骤声明为抽象方法或钩子方法。
4. 子类实现具体步骤。
5. 客户端调用模板方法。

## 使用场景

- 框架生命周期、数据导入导出、测试基类。
- Servlet 的 `service()` 调用 `doGet()`/`doPost()`。
- `JdbcTemplate`、`InputStream.read()` 等。

## 优点

- 复用算法骨架，减少重复代码。
- 流程受控，子类只能改指定步骤。
- 符合开闭原则。

## 缺点

- 依赖继承，灵活性有限。
- 父类变化会影响所有子类。
- 步骤过多时继承层次难以维护。

## Java 示例

```java
// 抽象类：定义饮品冲泡的固定流程
abstract class Beverage {
    // 模板方法：固定冲泡顺序，子类不能改写
    public final void prepare() {
        boilWater();
        brew();
        pourInCup();
        addCondiments();
        hook();
    }

    // 可变步骤：冲泡主料
    protected abstract void brew();

    // 可变步骤：添加配料
    protected abstract void addCondiments();

    // 固定步骤：烧水
    private void boilWater() {
        System.out.println("烧水");
    }

    // 固定步骤：倒入杯中
    private void pourInCup() {
        System.out.println("倒入杯中");
    }

    // 钩子方法：子类可以选择性覆盖
    protected void hook() {
    }
}

// 具体类：咖啡
class Coffee extends Beverage {
    // 冲泡咖啡粉
    @Override
    protected void brew() {
        System.out.println("冲泡咖啡粉");
    }

    // 加糖和牛奶
    @Override
    protected void addCondiments() {
        System.out.println("加糖和牛奶");
    }
}

// 具体类：茶
class Tea extends Beverage {
    // 浸泡茶叶
    @Override
    protected void brew() {
        System.out.println("浸泡茶叶");
    }

    // 加柠檬
    @Override
    protected void addCondiments() {
        System.out.println("加柠檬");
    }

    // 覆盖钩子，输出额外提醒
    @Override
    protected void hook() {
        System.out.println("提醒：茶要趁热喝");
    }
}

public class TemplateMethodDemo {
    // 程序入口：分别冲泡咖啡和茶
    public static void main(String[] args) {
        Beverage coffee = new Coffee();
        coffee.prepare();

        System.out.println("---");

        Beverage tea = new Tea();
        tea.prepare();
    }
}
```

---

[返回技能说明](../SKILL.md)
