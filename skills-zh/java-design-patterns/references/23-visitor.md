# 23. 访问者（Visitor）

> **分类**：行为型  
> **意图**：表示一个作用于某对象结构中的各元素的操作，使你不改变各元素的类就可以定义作用于这些元素的新操作。

## 使用说明

访问者模式把作用于对象结构的操作从元素类中分离出来。元素接口声明 `accept(visitor)`，每个元素在 `accept()` 中调用访问者对应的 `visit()` 方法，形成双重分派。新增操作只需增加访问者，不需要修改元素；但新增元素类型要修改所有访问者。

1. 定义访问者接口，为每种元素声明 `visit()` 重载。
2. 定义元素接口，声明 `accept(visitor)`。
3. 每个元素在 `accept()` 中调用 `visitor.visit(this)`。
4. 具体访问者实现操作。
5. 对象结构遍历元素并接受访问者。

## 使用场景

- AST/语法树遍历、编译器检查。
- 报表导出、对象结构上的多种统计。
- 对象结构稳定但操作频繁增加。

## 优点

- 新增操作容易，符合开闭原则。
- 相关操作集中在一个访问者中。
- 元素类保持稳定，职责清晰。

## 缺点

- 新增元素类型要修改所有访问者。
- 访问者可能需要元素暴露内部状态。
- 双重分派不直观，理解成本高。

## Java 示例

```java
// 访问者接口：为每种元素声明一个 visit 重载
interface ShapeVisitor {
    // 访问圆形
    void visit(Circle circle);

    // 访问矩形
    void visit(Rectangle rectangle);
}

// 元素接口：接受访问者
interface Shape {
    // 把自身交给访问者处理
    void accept(ShapeVisitor visitor);
}

// 具体元素：圆形
class Circle implements Shape {
    private final double radius;

    // 创建指定半径的圆
    public Circle(double radius) {
        this.radius = radius;
    }

    // 返回半径，供访问者读取
    public double getRadius() {
        return radius;
    }

    // 调用访问者的圆形处理方法
    @Override
    public void accept(ShapeVisitor visitor) {
        visitor.visit(this);
    }
}

// 具体元素：矩形
class Rectangle implements Shape {
    private final double width;
    private final double height;

    // 创建指定宽高的矩形
    public Rectangle(double width, double height) {
        this.width = width;
        this.height = height;
    }

    // 返回宽度，供访问者读取
    public double getWidth() {
        return width;
    }

    // 返回高度，供访问者读取
    public double getHeight() {
        return height;
    }

    // 调用访问者的矩形处理方法
    @Override
    public void accept(ShapeVisitor visitor) {
        visitor.visit(this);
    }
}

// 访问者：计算总面积
class AreaVisitor implements ShapeVisitor {
    private double total;

    // 累加圆面积
    @Override
    public void visit(Circle circle) {
        total += Math.PI * circle.getRadius() * circle.getRadius();
    }

    // 累加矩形面积
    @Override
    public void visit(Rectangle rectangle) {
        total += rectangle.getWidth() * rectangle.getHeight();
    }

    // 返回累计面积
    public double getTotal() {
        return total;
    }
}

// 访问者：导出形状描述
class DescriptionVisitor implements ShapeVisitor {
    // 输出圆形描述
    @Override
    public void visit(Circle circle) {
        System.out.printf("圆形，半径 %.2f%n", circle.getRadius());
    }

    // 输出矩形描述
    @Override
    public void visit(Rectangle rectangle) {
        System.out.printf("矩形，宽 %.2f，高 %.2f%n", rectangle.getWidth(), rectangle.getHeight());
    }
}

public class VisitorDemo {
    // 程序入口：让同一组元素接受两种访问者
    public static void main(String[] args) {
        Shape[] shapes = {
                new Circle(2),
                new Rectangle(3, 4)
        };

        AreaVisitor areaVisitor = new AreaVisitor();
        DescriptionVisitor descriptionVisitor = new DescriptionVisitor();

        for (Shape shape : shapes) {
            shape.accept(areaVisitor);
            shape.accept(descriptionVisitor);
        }

        System.out.printf("总面积：%.2f%n", areaVisitor.getTotal());
    }
}
```

---

[返回技能说明](../SKILL.md)
