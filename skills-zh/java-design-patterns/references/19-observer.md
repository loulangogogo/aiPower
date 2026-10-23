# 19. 观察者（Observer）

> **分类**：行为型  
> **意图**：定义对象间的一种一对多依赖关系，当一个对象的状态发生改变时，所有依赖它的对象都得到通知并自动更新。

## 使用说明

观察者模式定义一对多依赖：主题维护观察者列表，状态变化时通知所有观察者。观察者实现统一的更新接口，主题只依赖这个接口。Java 9 起 `java.util.Observer` 已废弃，实际项目可以自定义接口，或使用 Spring 事件、消息队列等机制。

1. 定义观察者接口。
2. 主题提供注册、注销和通知方法。
3. 主题维护观察者集合。
4. 状态变化时遍历通知。
5. 观察者实现更新逻辑，并及时注销。

## 使用场景

- 事件监听、发布订阅、MVC 视图更新。
- 订单状态变化通知多个下游。
- 配置变更广播。

## 优点

- 主题与观察者松耦合。
- 支持一对多广播。
- 新增观察者不需要修改主题。

## 缺点

- 通知顺序不确定。
- 可能引起连锁更新或重复通知。
- 忘记注销会导致内存泄漏。

## Java 示例

```java
import java.util.ArrayList;
import java.util.List;

// 观察者接口：主题状态变化时回调
interface Observer {
    // 接收主题推送的最新天气
    void update(String weather);
}

// 主题接口：管理观察者并发送通知
interface Subject {
    // 注册观察者
    void register(Observer observer);

    // 注销观察者
    void unregister(Observer observer);

    // 通知所有观察者
    void notifyObservers();
}

// 具体主题：气象站
class WeatherStation implements Subject {
    private final List<Observer> observers = new ArrayList<>();
    private String weather;

    // 注册观察者
    @Override
    public void register(Observer observer) {
        observers.add(observer);
    }

    // 注销观察者
    @Override
    public void unregister(Observer observer) {
        observers.remove(observer);
    }

    // 把当前天气推送给所有观察者
    @Override
    public void notifyObservers() {
        for (Observer observer : observers) {
            observer.update(weather);
        }
    }

    // 更新天气并触发通知
    public void setWeather(String weather) {
        this.weather = weather;
        System.out.println("气象站更新：" + weather);
        notifyObservers();
    }
}

// 具体观察者：显示终端
class PhoneDisplay implements Observer {
    private final String name;

    // 创建显示终端并命名
    public PhoneDisplay(String name) {
        this.name = name;
    }

    // 输出收到的天气
    @Override
    public void update(String weather) {
        System.out.println(name + " 收到天气：" + weather);
    }
}

public class ObserverDemo {
    // 程序入口：注册观察者、推送天气，再注销一个观察者
    public static void main(String[] args) {
        WeatherStation station = new WeatherStation();
        Observer phone = new PhoneDisplay("手机");
        Observer web = new PhoneDisplay("网页");

        station.register(phone);
        station.register(web);
        station.setWeather("晴，26℃");

        station.unregister(web);
        station.setWeather("小雨，18℃");
    }
}
```

---

[返回技能说明](../SKILL.md)
