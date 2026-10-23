# 08. 组合（Composite）

> **分类**：结构型  
> **意图**：将对象组合成树形结构来表示“部分-整体”的层次结构，使客户端可以一致地对待单个对象和组合对象。

## 使用说明

组合模式把对象组织成树形结构，叶子节点和容器节点实现同一个接口。容器节点持有子节点列表，并把操作递归委托给子节点。客户端不需要判断当前是叶子还是容器，统一调用接口即可。

1. 定义统一的组件接口。
2. 叶子节点实现基本操作。
3. 容器节点实现同一接口，并维护子节点集合。
4. 容器的操作递归调用子节点。
5. 客户端通过组件接口操作整棵树。

## 使用场景

- 文件与目录、组织架构、菜单树。
- UI 容器与控件。
- 需要对整棵树统一执行统计、渲染或校验。

## 优点

- 统一对待单个对象和组合对象。
- 树形结构表达自然，递归实现简洁。
- 新增节点类型比较容易。

## 缺点

- 接口过于一般化，难以限制容器里只能放某类节点。
- 递归深度大时可能栈溢出。
- 调试和排查递归逻辑较麻烦。

## Java 示例

```java
import java.util.ArrayList;
import java.util.List;

// 组件接口：叶子节点和容器节点都实现它
interface FileSystemNode {
    // 输出节点内容，indent 表示当前缩进
    void show(String indent);
}

// 叶子节点：普通文件
class FileNode implements FileSystemNode {
    private final String name;

    // 创建文件节点
    public FileNode(String name) {
        this.name = name;
    }

    // 输出文件名
    @Override
    public void show(String indent) {
        System.out.println(indent + "- " + name);
    }
}

// 容器节点：目录，内部持有子节点列表
class DirectoryNode implements FileSystemNode {
    private final String name;
    private final List<FileSystemNode> children = new ArrayList<>();

    // 创建目录节点
    public DirectoryNode(String name) {
        this.name = name;
    }

    // 向目录中添加文件或子目录
    public void add(FileSystemNode node) {
        children.add(node);
    }

    // 输出目录名，并递归输出所有子节点
    @Override
    public void show(String indent) {
        System.out.println(indent + "+ " + name + "/");
        for (FileSystemNode child : children) {
            child.show(indent + "  ");
        }
    }
}

public class CompositeDemo {
    // 程序入口：构建目录树并统一渲染
    public static void main(String[] args) {
        DirectoryNode root = new DirectoryNode("项目");
        DirectoryNode src = new DirectoryNode("src");

        src.add(new FileNode("Main.java"));
        src.add(new FileNode("Utils.java"));
        root.add(src);
        root.add(new FileNode("README.md"));

        root.show("");
    }
}
```

---

[返回技能说明](../SKILL.md)
