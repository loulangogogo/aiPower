# 17. Mediator

> **Category**: Behavioral  
> **Intent**: Define an object that encapsulates how a set of objects interact, so that the objects do not need to refer to each other explicitly.

## Usage

The mediator makes a group of objects stop referencing each other directly and instead communicate through the mediator. Colleague objects hold only the mediator, and the mediator maintains the colleague list and the interaction rules. This turns object relationships from a mesh into a star, and changing the interaction rules requires modifying only the mediator. Split the mediator in time when its responsibilities become too heavy.

1. Define the mediator interface.
2. Colleague objects depend only on the mediator.
3. The mediator maintains the colleague collection and implements forwarding rules.
4. Communication between colleagues is handed to the mediator.
5. Split the mediator by business area when it grows too large.

## Use Cases

- Chat rooms, linked UI controls, airport scheduling.
- API gateways, message buses, event buses.
- Multiple objects forming a mesh of dependencies.

## Pros

- Reduces direct coupling between objects.
- Interaction rules are centralized and easy to modify.
- Colleague objects can be reused independently.

## Cons

- The mediator can easily become a god class.
- Centralization brings complexity and performance pressure.
- Overuse makes the system hard to split apart.

## Java Example

```java
import java.util.ArrayList;
import java.util.List;

// Mediator interface: responsible for forwarding messages between colleagues
interface ChatMediator {
    // Add a chat-room member
    void addUser(User user);

    // Forward a message; from is the sender's name
    void sendMessage(String from, String message);
}

// Concrete mediator: chat room
class ChatRoom implements ChatMediator {
    private final List<User> users = new ArrayList<>();

    // Add a user to the chat room
    @Override
    public void addUser(User user) {
        users.add(user);
    }

    // Forward the message to every member except the sender
    @Override
    public void sendMessage(String from, String message) {
        for (User user : users) {
            if (!user.getName().equals(from)) {
                user.receive(from, message);
            }
        }
    }
}

// Colleague object: a user depends only on the mediator
class User {
    private final String name;
    private final ChatMediator mediator;

    // Create a user and bind it to the chat room
    public User(String name, ChatMediator mediator) {
        this.name = name;
        this.mediator = mediator;
    }

    // Return the user name
    public String getName() {
        return name;
    }

    // Send a message to the chat room
    public void send(String message) {
        System.out.println(name + " sends: " + message);
        mediator.sendMessage(name, message);
    }

    // Receive a message from another user
    public void receive(String from, String message) {
        System.out.println(name + " received a message from " + from + ": " + message);
    }
}

public class MediatorDemo {
    // Entry point: create a chat room and members, demonstrating message forwarding
    public static void main(String[] args) {
        ChatMediator room = new ChatRoom();
        User alice = new User("Alice", room);
        User bob = new User("Bob", room);
        User carol = new User("Carol", room);

        room.addUser(alice);
        room.addUser(bob);
        room.addUser(carol);

        alice.send("Hello everyone");
        bob.send("Hi, Alice");
    }
}
```

---

[Back to skill overview](../SKILL.md)
