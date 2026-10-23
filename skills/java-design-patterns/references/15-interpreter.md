# 15. Interpreter

> **Category**: Behavioral  
> **Intent**: Given a language, define a representation for its grammar along with an interpreter that uses the representation to interpret sentences in the language.

## Usage

The interpreter pattern defines a grammar for a simple language, with each grammar rule corresponding to an expression class. Terminal expressions handle basic elements, nonterminal expressions compose subexpressions, and the client builds the syntax tree and calls `interpret()`. It suits scenarios where the grammar is simple and changes infrequently; complex grammars should use parser generators such as ANTLR.

1. Define the expression interface and declare `interpret(context)`.
2. Implement an expression class for each terminal.
3. Implement composite expression classes for nonterminals.
4. The client builds the syntax tree according to the grammar.
5. Pass in the context and interpret recursively.

## Use Cases

- Simple rule engines, expression evaluation, query conditions.
- Custom DSLs, regular expressions, or SQL fragment interpretation.
- Grammar rules that need to be modeled explicitly as a class structure.

## Pros

- Grammar rules are clear and easy to extend.
- The implementation is intuitive and suits simple languages.
- Expressions can be composed and reused.

## Cons

- Complex grammars cause an explosion in the number of classes.
- Recursive interpretation has mediocre performance.
- Maintenance and debugging costs are relatively high.

## Java Example

```java
import java.util.HashMap;
import java.util.Map;

// Expression interface: every syntax node implements the interpretation method
interface Expression {
    // Compute the integer value of the expression from the context
    int interpret(Map<String, Integer> context);
}

// Terminal expression: number
class NumberExpression implements Expression {
    private final int number;

    // Store the numeric literal
    public NumberExpression(int number) {
        this.number = number;
    }

    // A number returns itself directly
    @Override
    public int interpret(Map<String, Integer> context) {
        return number;
    }
}

// Terminal expression: variable
class VariableExpression implements Expression {
    private final String name;

    // Store the variable name
    public VariableExpression(String name) {
        this.name = name;
    }

    // Look up the variable value in the context
    @Override
    public int interpret(Map<String, Integer> context) {
        Integer value = context.get(name);
        if (value == null) {
            throw new IllegalArgumentException("Undefined variable: " + name);
        }
        return value;
    }
}

// Nonterminal expression: addition
class AddExpression implements Expression {
    private final Expression left;
    private final Expression right;

    // Store the left and right subexpressions
    public AddExpression(Expression left, Expression right) {
        this.left = left;
        this.right = right;
    }

    // Interpret the left and right expressions separately, then add them
    @Override
    public int interpret(Map<String, Integer> context) {
        return left.interpret(context) + right.interpret(context);
    }
}

// Nonterminal expression: subtraction
class SubtractExpression implements Expression {
    private final Expression left;
    private final Expression right;

    // Store the left and right subexpressions
    public SubtractExpression(Expression left, Expression right) {
        this.left = left;
        this.right = right;
    }

    // Interpret the left and right expressions separately, then subtract them
    @Override
    public int interpret(Map<String, Integer> context) {
        return left.interpret(context) - right.interpret(context);
    }
}

public class InterpreterDemo {
    // Entry point: build the syntax tree for (a + b) - 10 and evaluate it
    public static void main(String[] args) {
        Expression expression = new SubtractExpression(
                new AddExpression(new VariableExpression("a"), new VariableExpression("b")),
                new NumberExpression(10)
        );

        Map<String, Integer> context = new HashMap<>();
        context.put("a", 30);
        context.put("b", 12);

        System.out.println("Result: " + expression.interpret(context));
    }
}
```

---

[Back to skill overview](../SKILL.md)
