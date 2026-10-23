# 17. 中介者（Mediator）

> **分类**：行为型  
> **意图**：用一个中介对象来封装一系列对象之间的交互，使各对象不需要显式地相互引用。

## 使用说明

中介者让一组对象不再直接互相引用，而是统一通过中介者通信。同事对象只持有中介者，中介者维护同事列表和交互规则。这样对象之间的关系从网状变成星形，修改交互规则只需要改中介者。中介者职责过重时要及时拆分。

1. 定义中介者接口。
2. 同事对象只依赖中介者。
3. 中介者维护同事集合并实现转发规则。
4. 同事之间的通信交给中介者。
5. 中介者膨胀时按业务拆分。

## 使用场景

- 聊天室、UI 控件联动、机场调度。
- API 网关、消息总线、事件总线。
- 多个对象之间形成网状依赖。

## 优点

- 减少对象之间的直接耦合。
- 交互规则集中，便于修改。
- 同事对象可以独立复用。

## 缺点

- 中介者容易变成上帝类。
- 中心化带来复杂度和性能压力。
- 过度使用会让系统难以拆分。

## Java 示例

```java
import java.util.ArrayList;
import java.util.List;

// 中介者接口：负责同事之间的消息转发
interface ChatMediator {
    // 添加聊天室成员
    void addUser(User user);

    // 转发消息，from 表示发送者名称
    void sendMessage(String from, String message);
}

// 具体中介者：聊天室
class ChatRoom implements ChatMediator {
    private final List<User> users = new ArrayList<>();

    // 将用户加入聊天室
    @Override
    public void addUser(User user) {
        users.add(user);
    }

    // 将消息转发给除发送者以外的所有成员
    @Override
    public void sendMessage(String from, String message) {
        for (User user : users) {
            if (!user.getName().equals(from)) {
                user.receive(from, message);
            }
        }
    }
}

// 同事对象：用户只依赖中介者
class User {
    private final String name;
    private final ChatMediator mediator;

    // 创建用户并绑定聊天室
    public User(String name, ChatMediator mediator) {
        this.name = name;
        this.mediator = mediator;
    }

    // 返回用户名
    public String getName() {
        return name;
    }

    // 发送消息给聊天室
    public void send(String message) {
        System.out.println(name + " 发送：" + message);
        mediator.sendMessage(name, message);
    }

    // 接收其他用户的消息
    public void receive(String from, String message) {
        System.out.println(name + " 收到 " + from + " 的消息：" + message);
    }
}

public class MediatorDemo {
    // 程序入口：创建聊天室和成员，演示消息转发
    public static void main(String[] args) {
        ChatMediator room = new ChatRoom();
        User alice = new User("Alice", room);
        User bob = new User("Bob", room);
        User carol = new User("Carol", room);

        room.addUser(alice);
        room.addUser(bob);
        room.addUser(carol);

        alice.send("大家好");
        bob.send("你好，Alice");
    }
}
```

---

[返回技能说明](../SKILL.md)
