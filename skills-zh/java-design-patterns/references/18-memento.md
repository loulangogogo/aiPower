# 18. 备忘录（Memento）

> **分类**：行为型  
> **意图**：在不破坏封装性的前提下，捕获一个对象的内部状态，并在该对象之外保存这个状态，以便以后恢复。

## 使用说明

备忘录模式把对象的内部状态保存到外部，以便之后恢复。发起人负责创建和恢复快照，备忘录只保存状态且对外只读，管理者负责保存快照但不修改内容。快照数量多时要考虑内存，可以限制历史长度或使用增量快照。

1. 定义备忘录类，保存状态并提供只读访问。
2. 发起人提供 `save()` 和 `restore()`。
3. 管理者保存备忘录栈或列表。
4. 需要时把备忘录交还给发起人恢复。
5. 控制历史数量和快照大小。

## 使用场景

- 编辑器撤销/重做、表单草稿、游戏存档。
- 事务回滚、流程回退。
- 需要保存对象历史状态。

## 优点

- 不破坏封装即可保存状态。
- 恢复逻辑简单。
- 历史管理集中在管理者。

## 缺点

- 快照占用内存。
- 频繁快照影响性能。
- 管理者生命周期需要管理。

## Java 示例

```java
import java.util.ArrayDeque;
import java.util.Deque;

// 备忘录：只保存状态，不提供修改方法
final class EditorMemento {
    private final String content;

    // 创建快照并保存内容
    EditorMemento(String content) {
        this.content = content;
    }

    // 返回快照中的内容
    String getContent() {
        return content;
    }
}

// 发起人：负责创建和恢复快照
class Editor {
    private String content = "";

    // 追加输入内容
    public void type(String text) {
        content += text;
    }

    // 返回当前内容
    public String getContent() {
        return content;
    }

    // 创建当前状态的快照
    public EditorMemento save() {
        return new EditorMemento(content);
    }

    // 根据快照恢复内容
    public void restore(EditorMemento memento) {
        this.content = memento.getContent();
    }
}

// 管理者：只负责保存快照，不关心内容
class History {
    private final Deque<EditorMemento> stack = new ArrayDeque<>();

    // 保存一个快照
    public void push(EditorMemento memento) {
        stack.push(memento);
    }

    // 取出最近的快照
    public EditorMemento pop() {
        return stack.pop();
    }
}

public class MementoDemo {
    // 程序入口：编辑文本，保存快照并逐步撤销
    public static void main(String[] args) {
        Editor editor = new Editor();
        History history = new History();

        editor.type("第一版");
        history.push(editor.save());

        editor.type("，第二版");
        history.push(editor.save());

        editor.type("，第三版");
        System.out.println("当前内容：" + editor.getContent());

        editor.restore(history.pop());
        System.out.println("撤销一次：" + editor.getContent());

        editor.restore(history.pop());
        System.out.println("再撤销一次：" + editor.getContent());
    }
}
```

---

[返回技能说明](../SKILL.md)
