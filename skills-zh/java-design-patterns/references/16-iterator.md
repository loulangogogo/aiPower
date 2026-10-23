# 16. 迭代器（Iterator）

> **分类**：行为型  
> **意图**：提供一种方法顺序访问一个聚合对象中的各个元素，而又不暴露该对象的内部表示。

## 使用说明

迭代器把遍历逻辑从聚合对象中抽出来。聚合对象负责提供迭代器，迭代器保存当前位置并实现 `hasNext()` 和 `next()`。Java 中让集合实现 `Iterable<T>`，就可以直接使用 for-each。遍历期间修改集合要使用迭代器自己的 `remove()` 或并发集合。

1. 定义迭代器接口，包含 `hasNext()` 和 `next()`。
2. 聚合对象提供创建迭代器的方法。
3. 迭代器内部保存遍历位置。
4. 客户端通过迭代器访问元素。
5. 处理越界和并发修改。

## 使用场景

- 自定义集合、树/图遍历、分页结果。
- 需要多种遍历方式或延迟加载。
- 不想暴露集合内部结构。

## 优点

- 统一遍历接口。
- 遍历算法与聚合结构分离。
- 支持多种遍历方式和多个迭代器。

## 缺点

- 简单集合使用会显得繁琐。
- 并发修改需要额外处理。
- 迭代器生命周期需要管理。

## Java 示例

```java
import java.util.ArrayList;
import java.util.Iterator;
import java.util.List;
import java.util.NoSuchElementException;

// 集合中的元素
class Book {
    private final String title;

    // 创建一本书
    public Book(String title) {
        this.title = title;
    }

    // 返回书名
    public String getTitle() {
        return title;
    }
}

// 聚合对象：实现 Iterable，支持 for-each 遍历
class BookShelf implements Iterable<Book> {
    private final List<Book> books = new ArrayList<>();

    // 向书架添加一本书
    public void add(Book book) {
        books.add(book);
    }

    // 创建书架专属的迭代器
    @Override
    public Iterator<Book> iterator() {
        return new BookIterator();
    }

    // 内部迭代器：直接访问 BookShelf 的私有数据
    private class BookIterator implements Iterator<Book> {
        private int index = 0;

        // 判断是否还有下一个元素
        @Override
        public boolean hasNext() {
            return index < books.size();
        }

        // 返回下一个元素并移动游标
        @Override
        public Book next() {
            if (!hasNext()) {
                throw new NoSuchElementException("没有更多元素");
            }
            return books.get(index++);
        }
    }
}

public class IteratorDemo {
    // 程序入口：使用 for-each 遍历自定义集合
    public static void main(String[] args) {
        BookShelf shelf = new BookShelf();
        shelf.add(new Book("深入理解 Java 虚拟机"));
        shelf.add(new Book("Effective Java"));
        shelf.add(new Book("设计模式"));

        for (Book book : shelf) {
            System.out.println(book.getTitle());
        }
    }
}
```

---

[返回技能说明](../SKILL.md)
