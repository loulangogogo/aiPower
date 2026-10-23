# 03. 抽象工厂（Abstract Factory）

> **分类**：创建型  
> **意图**：提供一个创建一系列相关或相互依赖对象的接口，而不指定它们的具体类。

## 使用说明

抽象工厂面向“产品族”：一个工厂负责创建一组相关产品。客户端只依赖抽象工厂和抽象产品，切换整套实现时只需替换工厂。它和工厂方法的区别在于，工厂方法通常只创建一个产品，抽象工厂创建一族产品。

1. 为每个产品种类定义接口。
2. 定义抽象工厂，声明一族产品的创建方法。
3. 为每个产品族实现具体工厂，保证同族产品搭配使用。
4. 客户端只使用抽象工厂和抽象产品。

## 使用场景

- 跨平台 UI：Windows 与 macOS 的按钮、复选框、菜单。
- 多套主题、多数据库方言、多环境客户端。
- 必须保证同一族对象搭配使用，不能混用。

## 优点

- 保证产品族内对象的一致性。
- 客户端与具体实现解耦。
- 切换产品族只需替换工厂。

## 缺点

- 新增产品种类要修改所有工厂，违反开闭原则。
- 类的数量明显增加。
- 抽象层较重，小项目容易过度设计。

## Java 示例

```java
// 产品族中的按钮接口
interface Button {
    // 渲染按钮
    void render();
}

// 产品族中的复选框接口
interface Checkbox {
    // 渲染复选框
    void render();
}

// Windows 风格按钮
class WindowsButton implements Button {
    // 输出 Windows 按钮的渲染结果
    @Override
    public void render() {
        System.out.println("渲染 Windows 按钮");
    }
}

// Windows 风格复选框
class WindowsCheckbox implements Checkbox {
    // 输出 Windows 复选框的渲染结果
    @Override
    public void render() {
        System.out.println("渲染 Windows 复选框");
    }
}

// macOS 风格按钮
class MacButton implements Button {
    // 输出 macOS 按钮的渲染结果
    @Override
    public void render() {
        System.out.println("渲染 macOS 按钮");
    }
}

// macOS 风格复选框
class MacCheckbox implements Checkbox {
    // 输出 macOS 复选框的渲染结果
    @Override
    public void render() {
        System.out.println("渲染 macOS 复选框");
    }
}

// 抽象工厂：声明创建一族 UI 组件的方法
interface UiFactory {
    // 创建按钮
    Button createButton();

    // 创建复选框
    Checkbox createCheckbox();
}

// Windows 产品族工厂
class WindowsFactory implements UiFactory {
    // 创建 Windows 按钮
    @Override
    public Button createButton() {
        return new WindowsButton();
    }

    // 创建 Windows 复选框
    @Override
    public Checkbox createCheckbox() {
        return new WindowsCheckbox();
    }
}

// macOS 产品族工厂
class MacFactory implements UiFactory {
    // 创建 macOS 按钮
    @Override
    public Button createButton() {
        return new MacButton();
    }

    // 创建 macOS 复选框
    @Override
    public Checkbox createCheckbox() {
        return new MacCheckbox();
    }
}

public class AbstractFactoryDemo {
    // 程序入口：使用同一个工厂创建同一族组件
    public static void main(String[] args) {
        UiFactory factory = new MacFactory();
        Button button = factory.createButton();
        Checkbox checkbox = factory.createCheckbox();

        button.render();
        checkbox.render();
    }
}
```

---

[返回技能说明](../SKILL.md)
