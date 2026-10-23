# 16. Iterator

> **Category**: Behavioral  
> **Intent**: Provide a way to access the elements of an aggregate object sequentially without exposing its underlying representation.

## Usage

The iterator extracts traversal logic from the aggregate object. The aggregate is responsible for providing an iterator; the iterator stores the current position and implements `hasNext()` and `next()`. In Java, making a collection implement `Iterable<T>` allows for-each to be used directly. To modify a collection during iteration, use the iterator's own `remove()` or a concurrent collection.

1. Define the iterator interface, including `hasNext()` and `next()`.
2. The aggregate object provides a method that creates an iterator.
3. The iterator stores the traversal position internally.
4. The client accesses elements through the iterator.
5. Handle out-of-bounds access and concurrent modification.

## Use Cases

- Custom collections, tree/graph traversal, paginated results.
- Multiple traversal styles or lazy loading.
- A collection whose internal structure should not be exposed.

## Pros

- Unified traversal interface.
- Separates traversal algorithms from the aggregate structure.
- Supports multiple traversal styles and multiple iterators.

## Cons

- Using it for simple collections feels cumbersome.
- Concurrent modification requires extra handling.
- Iterator lifecycle must be managed.

## Java Example

```java
import java.util.ArrayList;
import java.util.Iterator;
import java.util.List;
import java.util.NoSuchElementException;

// Element in the collection
class Book {
    private final String title;

    // Create a book
    public Book(String title) {
        this.title = title;
    }

    // Return the book title
    public String getTitle() {
        return title;
    }
}

// Aggregate object: implements Iterable to support for-each traversal
class BookShelf implements Iterable<Book> {
    private final List<Book> books = new ArrayList<>();

    // Add a book to the shelf
    public void add(Book book) {
        books.add(book);
    }

    // Create an iterator dedicated to the bookshelf
    @Override
    public Iterator<Book> iterator() {
        return new BookIterator();
    }

    // Inner iterator: directly accesses the private data of BookShelf
    private class BookIterator implements Iterator<Book> {
        private int index = 0;

        // Determine whether there is a next element
        @Override
        public boolean hasNext() {
            return index < books.size();
        }

        // Return the next element and advance the cursor
        @Override
        public Book next() {
            if (!hasNext()) {
                throw new NoSuchElementException("No more elements");
            }
            return books.get(index++);
        }
    }
}

public class IteratorDemo {
    // Entry point: traverse a custom collection with for-each
    public static void main(String[] args) {
        BookShelf shelf = new BookShelf();
        shelf.add(new Book("Understanding the Java Virtual Machine"));
        shelf.add(new Book("Effective Java"));
        shelf.add(new Book("Design Patterns"));

        for (Book book : shelf) {
            System.out.println(book.getTitle());
        }
    }
}
```

---

[Back to skill overview](../SKILL.md)
