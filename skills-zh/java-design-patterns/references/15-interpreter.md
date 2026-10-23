# 15. 解释器（Interpreter）

> **分类**：行为型  
> **意图**：给定一个语言，定义它的文法表示，并定义一个解释器来解释该语言中的句子。

## 使用说明

解释器模式为一种简单语言定义文法，每个文法规则对应一个表达式类。终结符表达式处理基本元素，非终结符表达式组合子表达式，客户端构造语法树后调用 `interpret()`。它适合文法简单、变化不频繁的场景；复杂文法应使用 ANTLR 等解析器生成器。

1. 定义表达式接口，声明 `interpret(context)`。
2. 为终结符实现表达式类。
3. 为非终结符实现组合表达式类。
4. 客户端按文法构造语法树。
5. 传入上下文并递归解释。

## 使用场景

- 简单规则引擎、表达式计算、查询条件。
- 自定义 DSL、正则或 SQL 片段解释。
- 需要把语法规则显式建模为类结构。

## 优点

- 文法规则清晰，易于扩展。
- 实现直观，适合简单语言。
- 表达式可以组合和复用。

## 缺点

- 复杂文法会导致类数量爆炸。
- 递归解释的性能一般。
- 维护和调试成本较高。

## Java 示例

```java
import java.util.HashMap;
import java.util.Map;

// 表达式接口：所有语法节点都实现解释方法
interface Expression {
    // 根据上下文计算表达式的整数值
    int interpret(Map<String, Integer> context);
}

// 终结符表达式：数字
class NumberExpression implements Expression {
    private final int number;

    // 保存数字字面量
    public NumberExpression(int number) {
        this.number = number;
    }

    // 数字直接返回自身
    @Override
    public int interpret(Map<String, Integer> context) {
        return number;
    }
}

// 终结符表达式：变量
class VariableExpression implements Expression {
    private final String name;

    // 保存变量名
    public VariableExpression(String name) {
        this.name = name;
    }

    // 从上下文中查找变量值
    @Override
    public int interpret(Map<String, Integer> context) {
        Integer value = context.get(name);
        if (value == null) {
            throw new IllegalArgumentException("未定义变量：" + name);
        }
        return value;
    }
}

// 非终结符表达式：加法
class AddExpression implements Expression {
    private final Expression left;
    private final Expression right;

    // 保存左右两个子表达式
    public AddExpression(Expression left, Expression right) {
        this.left = left;
        this.right = right;
    }

    // 分别解释左右表达式后相加
    @Override
    public int interpret(Map<String, Integer> context) {
        return left.interpret(context) + right.interpret(context);
    }
}

// 非终结符表达式：减法
class SubtractExpression implements Expression {
    private final Expression left;
    private final Expression right;

    // 保存左右两个子表达式
    public SubtractExpression(Expression left, Expression right) {
        this.left = left;
        this.right = right;
    }

    // 分别解释左右表达式后相减
    @Override
    public int interpret(Map<String, Integer> context) {
        return left.interpret(context) - right.interpret(context);
    }
}

public class InterpreterDemo {
    // 程序入口：构造 (a + b) - 10 的语法树并计算
    public static void main(String[] args) {
        Expression expression = new SubtractExpression(
                new AddExpression(new VariableExpression("a"), new VariableExpression("b")),
                new NumberExpression(10)
        );

        Map<String, Integer> context = new HashMap<>();
        context.put("a", 30);
        context.put("b", 12);

        System.out.println("计算结果：" + expression.interpret(context));
    }
}
```

---

[返回技能说明](../SKILL.md)
