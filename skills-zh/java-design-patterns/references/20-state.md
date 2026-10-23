# 20. 状态（State）

> **分类**：行为型  
> **意图**：允许一个对象在其内部状态改变时改变它的行为，对象看起来似乎修改了它的类。

## 使用说明

状态模式把对象在不同状态下的行为封装成独立的类。上下文持有当前状态，并把请求委托给状态对象；状态对象在合适的时机触发状态转换。与策略模式不同，状态通常由内部事件驱动自动转换，客户端一般不需要直接选择状态。

1. 定义状态接口，声明该状态支持的操作。
2. 每个具体状态实现行为。
3. 上下文持有当前状态并委托请求。
4. 在状态对象或上下文中完成状态转换。
5. 集中梳理状态转换图，避免遗漏。

## 使用场景

- 订单状态：待支付、已支付、已发货、已取消。
- 电梯、红绿灯、播放器、工作流引擎。
- 需要消除大量与状态相关的 if/switch。

## 优点

- 消除状态判断的条件分支。
- 每个状态职责单一。
- 状态转换清晰，易于扩展新状态。

## 缺点

- 状态类数量增加。
- 转换分散时难以把握全局。
- 简单状态机使用会过度设计。

## Java 示例

```java
// 状态接口：声明订单在各状态下支持的操作
interface OrderState {
    // 支付订单
    void pay(OrderContext order);

    // 发货
    void ship(OrderContext order);

    // 取消订单
    void cancel(OrderContext order);
}

// 待支付状态
class PendingPaymentState implements OrderState {
    // 支付成功后切换到已支付状态
    @Override
    public void pay(OrderContext order) {
        System.out.println("支付成功");
        order.setState(new PaidState());
    }

    // 未支付时禁止发货
    @Override
    public void ship(OrderContext order) {
        System.out.println("未支付，不能发货");
    }

    // 取消订单并切换到已取消状态
    @Override
    public void cancel(OrderContext order) {
        System.out.println("订单已取消");
        order.setState(new CancelledState());
    }
}

// 已支付状态
class PaidState implements OrderState {
    // 已支付时禁止重复支付
    @Override
    public void pay(OrderContext order) {
        System.out.println("订单已支付，请勿重复支付");
    }

    // 发货后切换到已发货状态
    @Override
    public void ship(OrderContext order) {
        System.out.println("订单已发货");
        order.setState(new ShippedState());
    }

    // 已支付订单不能直接取消
    @Override
    public void cancel(OrderContext order) {
        System.out.println("已支付订单请走退款流程");
    }
}

// 已发货状态
class ShippedState implements OrderState {
    // 已发货时禁止支付
    @Override
    public void pay(OrderContext order) {
        System.out.println("订单已发货，不能支付");
    }

    // 已发货时禁止重复发货
    @Override
    public void ship(OrderContext order) {
        System.out.println("订单已发货，请勿重复操作");
    }

    // 已发货订单不能直接取消
    @Override
    public void cancel(OrderContext order) {
        System.out.println("已发货订单不能直接取消");
    }
}

// 已取消状态
class CancelledState implements OrderState {
    // 已取消时禁止支付
    @Override
    public void pay(OrderContext order) {
        System.out.println("订单已取消，不能支付");
    }

    // 已取消时禁止发货
    @Override
    public void ship(OrderContext order) {
        System.out.println("订单已取消，不能发货");
    }

    // 重复取消时给出提示
    @Override
    public void cancel(OrderContext order) {
        System.out.println("订单已取消");
    }
}

// 上下文：持有当前状态并把请求委托给状态对象
class OrderContext {
    private OrderState state = new PendingPaymentState();

    // 切换当前状态
    public void setState(OrderState state) {
        this.state = state;
    }

    // 发起支付请求
    public void pay() {
        state.pay(this);
    }

    // 发起发货请求
    public void ship() {
        state.ship(this);
    }

    // 发起取消请求
    public void cancel() {
        state.cancel(this);
    }
}

public class StateDemo {
    // 程序入口：依次执行支付、发货、取消，观察状态变化
    public static void main(String[] args) {
        OrderContext order = new OrderContext();
        order.pay();
        order.ship();
        order.cancel();
    }
}
```

---

[返回技能说明](../SKILL.md)
