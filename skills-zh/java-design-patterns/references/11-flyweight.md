# 11. 享元（Flyweight）

> **分类**：结构型  
> **意图**：运用共享技术有效地支持大量细粒度对象。

## 使用说明

享元模式把对象状态拆成内部状态和外部状态。内部状态可以共享，放在享元对象里并保持不可变；外部状态随使用场景变化，由客户端保存或传入。工厂用缓存复用相同内部状态的对象，从而减少对象数量。

1. 区分内部状态和外部状态。
2. 定义享元对象，只保存内部状态，并设为不可变。
3. 创建享元工厂，用 Map 缓存享元。
4. 客户端保存外部状态，使用时传给享元。
5. 注意并发访问和缓存清理。

## 使用场景

- 文本编辑器的字符样式、棋盘棋子、地图标记。
- 大量重复对象造成内存压力。
- 对象的大部分状态可以共享。

## 优点

- 显著减少对象数量和内存占用。
- 共享结构清晰，便于统一管理。
- 外部状态灵活，不污染共享对象。

## 缺点

- 增加工厂和查找开销。
- 内外状态划分需要仔细设计。
- 共享对象不可变，灵活性受限。

## Java 示例

```java
import java.util.HashMap;
import java.util.Map;

// 享元：树的内部状态，可共享且不可变
class TreeType {
    private final String name;
    private final String color;

    // 创建树类型，内部状态只在工厂中初始化
    public TreeType(String name, String color) {
        this.name = name;
        this.color = color;
    }

    // 根据外部传入的位置绘制树
    public void draw(int x, int y) {
        System.out.println("在 (" + x + ", " + y + ") 绘制 " + color + " 的 " + name);
    }
}

// 享元工厂：相同内部状态只创建一个对象
class TreeFactory {
    private static final Map<String, TreeType> CACHE = new HashMap<>();

    // 根据名称和颜色获取享元，不存在时创建并缓存
    public static TreeType getTreeType(String name, String color) {
        String key = name + "|" + color;
        return CACHE.computeIfAbsent(key, k -> new TreeType(name, color));
    }
}

// 外部状态：位置，由客户端保存
class Tree {
    private final int x;
    private final int y;
    private final TreeType type;

    // 创建树，组合外部位置和共享的树类型
    public Tree(int x, int y, TreeType type) {
        this.x = x;
        this.y = y;
        this.type = type;
    }

    // 使用共享的树类型绘制当前位置
    public void draw() {
        type.draw(x, y);
    }
}

public class FlyweightDemo {
    // 程序入口：大量树共享少量树类型
    public static void main(String[] args) {
        TreeType ginkgo = TreeFactory.getTreeType("银杏", "绿色");
        TreeType another = TreeFactory.getTreeType("银杏", "绿色");

        System.out.println("是否为同一个享元对象：" + (ginkgo == another));

        for (int i = 0; i < 3; i++) {
            new Tree(i * 10, i * 20, ginkgo).draw();
        }
    }
}
```

---

[返回技能说明](../SKILL.md)
