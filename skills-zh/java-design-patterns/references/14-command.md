# 14. 命令（Command）

> **分类**：行为型  
> **意图**：将一个请求封装为一个对象，从而使你可用不同的请求对客户进行参数化，支持排队、记录日志和撤销操作。

## 使用说明

命令模式把请求封装成对象。命令接口声明 `execute()`，具体命令持有接收者并调用它的业务方法；调用者只持有命令，不关心接收者是谁。需要撤销时，命令还要保存执行前的状态并实现 `undo()`。多个命令可以用组合模式打包成宏命令。

1. 定义命令接口，声明 `execute()`，可选 `undo()`。
2. 具体命令持有接收者，在 `execute()` 中调用接收者。
3. 调用者持有命令并触发执行。
4. 客户端创建命令并装配到调用者。
5. 需要撤销时保存历史状态。

## 使用场景

- 按钮、菜单、快捷键动作。
- 任务队列、异步执行、延迟执行。
- 撤销/重做、事务脚本、宏录制。

## 优点

- 调用者与接收者解耦。
- 支持排队、记录、撤销和组合。
- 命令对象可以复用和参数化。

## 缺点

- 每个操作都要一个命令类，类数量增加。
- 撤销逻辑实现复杂。
- 简单场景下会增加间接层。

## Java 示例

```java
// 命令接口：声明执行和撤销
interface Command {
    // 执行命令
    void execute();

    // 撤销命令
    void undo();
}

// 接收者：灯
class Light {
    // 打开灯
    public void on() {
        System.out.println("灯已打开");
    }

    // 关闭灯
    public void off() {
        System.out.println("灯已关闭");
    }
}

// 开灯命令
class LightOnCommand implements Command {
    private final Light light;

    // 注入灯对象
    public LightOnCommand(Light light) {
        this.light = light;
    }

    // 执行开灯
    @Override
    public void execute() {
        light.on();
    }

    // 撤销开灯：关灯
    @Override
    public void undo() {
        light.off();
    }
}

// 关灯命令
class LightOffCommand implements Command {
    private final Light light;

    // 注入灯对象
    public LightOffCommand(Light light) {
        this.light = light;
    }

    // 执行关灯
    @Override
    public void execute() {
        light.off();
    }

    // 撤销关灯：开灯
    @Override
    public void undo() {
        light.on();
    }
}

// 调用者：遥控器只持有命令，不关心命令如何执行
class RemoteControl {
    private Command command;

    // 设置当前按钮绑定的命令
    public void setCommand(Command command) {
        this.command = command;
    }

    // 按下按钮，执行当前命令
    public void press() {
        command.execute();
    }

    // 按下撤销按钮，撤销当前命令
    public void pressUndo() {
        command.undo();
    }
}

public class CommandDemo {
    // 程序入口：把命令装配到遥控器并执行
    public static void main(String[] args) {
        Light light = new Light();
        RemoteControl remote = new RemoteControl();

        remote.setCommand(new LightOnCommand(light));
        remote.press();
        remote.pressUndo();

        remote.setCommand(new LightOffCommand(light));
        remote.press();
    }
}
```

---

[返回技能说明](../SKILL.md)
