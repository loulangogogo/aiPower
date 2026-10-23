# 05. 原型（Prototype）

> **分类**：创建型  
> **意图**：用原型实例指定创建对象的种类，并通过复制这个原型创建新对象。

## 使用说明

原型模式通过复制已有对象来创建新对象，而不是重新走一遍构造流程。实现时让类实现 `Cloneable` 并重写 `clone()`，先调用 `super.clone()` 做浅拷贝，再对引用类型字段手动深拷贝。对象结构复杂时，拷贝构造器或序列化往往比 `clone()` 更可控。

1. 让原型类实现 `Cloneable`。
2. 重写 `clone()`，调用 `super.clone()`。
3. 对可变引用字段做深拷贝。
4. 客户端复制原型，再修改差异部分。
5. 注意循环引用和不可变对象，避免共享可变状态。

## 使用场景

- 对象创建成本高，例如需要查库、计算或远程调用。
- 需要保存对象在某个时刻的快照。
- 大量对象只有少量属性不同。

## 优点

- 复制通常比重新创建更快。
- 可以在运行时增加或删除原型。
- 简化相似对象的创建。

## 缺点

- 深拷贝实现复杂，循环引用难处理。
- `Cloneable` 是标记接口，设计上容易误用。
- 每个类都要实现克隆逻辑，维护成本高。

## Java 示例

```java
import java.util.ArrayList;
import java.util.List;

// 原型类：实现 Cloneable，支持复制自身
class Resume implements Cloneable {
    private String name;
    private List<String> skills;

    // 创建简历，技能列表做一次防御性复制
    public Resume(String name, List<String> skills) {
        this.name = name;
        this.skills = new ArrayList<>(skills);
    }

    // 复制简历：先浅拷贝，再单独复制引用类型字段
    @Override
    public Resume clone() {
        try {
            Resume copy = (Resume) super.clone();
            // List 是可变引用，必须新建列表，避免两个对象共享同一份数据
            copy.skills = new ArrayList<>(this.skills);
            return copy;
        } catch (CloneNotSupportedException e) {
            throw new AssertionError("Resume 未实现 Cloneable", e);
        }
    }

    // 向简历追加一项技能
    public void addSkill(String skill) {
        skills.add(skill);
    }

    // 输出简历内容，便于对比原对象和副本
    public void print() {
        System.out.println(name + " 的技能：" + skills);
    }
}

public class PrototypeDemo {
    // 程序入口：复制原型后修改副本，验证两者互不影响
    public static void main(String[] args) {
        Resume original = new Resume("张三", List.of("Java", "MySQL"));
        Resume copy = original.clone();

        copy.addSkill("Redis");

        original.print();
        copy.print();
    }
}
```

---

[返回技能说明](../SKILL.md)
