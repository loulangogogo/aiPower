# 19. Observer

> **Category**: Behavioral  
> **Intent**: Define a one-to-many dependency between objects so that when one object changes state, all its dependents are notified and updated automatically.

## Usage

The observer pattern defines a one-to-many dependency: the subject maintains a list of observers and notifies all observers when its state changes. Observers implement a unified update interface, and the subject depends only on that interface. Since Java 9, `java.util.Observer` has been deprecated; real projects can define their own interface or use mechanisms such as Spring events and message queues.

1. Define the observer interface.
2. The subject provides register, unregister, and notify methods.
3. The subject maintains a collection of observers.
4. Traverse and notify when the state changes.
5. Observers implement update logic and unregister in time.

## Use Cases

- Event listeners, publish/subscribe, MVC view updates.
- Order status changes notifying multiple downstream systems.
- Configuration change broadcasts.

## Pros

- Loose coupling between the subject and observers.
- Supports one-to-many broadcasting.
- Adding observers does not require modifying the subject.

## Cons

- Notification order is not guaranteed.
- May cause cascading updates or duplicate notifications.
- Forgetting to unregister causes memory leaks.

## Java Example

```java
import java.util.ArrayList;
import java.util.List;

// Observer interface: called back when the subject's state changes
interface Observer {
    // Receive the latest weather pushed by the subject
    void update(String weather);
}

// Subject interface: manages observers and sends notifications
interface Subject {
    // Register an observer
    void register(Observer observer);

    // Unregister an observer
    void unregister(Observer observer);

    // Notify all observers
    void notifyObservers();
}

// Concrete subject: weather station
class WeatherStation implements Subject {
    private final List<Observer> observers = new ArrayList<>();
    private String weather;

    // Register an observer
    @Override
    public void register(Observer observer) {
        observers.add(observer);
    }

    // Unregister an observer
    @Override
    public void unregister(Observer observer) {
        observers.remove(observer);
    }

    // Push the current weather to all observers
    @Override
    public void notifyObservers() {
        for (Observer observer : observers) {
            observer.update(weather);
        }
    }

    // Update the weather and trigger a notification
    public void setWeather(String weather) {
        this.weather = weather;
        System.out.println("Weather station update: " + weather);
        notifyObservers();
    }
}

// Concrete observer: display terminal
class PhoneDisplay implements Observer {
    private final String name;

    // Create a display terminal and name it
    public PhoneDisplay(String name) {
        this.name = name;
    }

    // Print the received weather
    @Override
    public void update(String weather) {
        System.out.println(name + " received weather: " + weather);
    }
}

public class ObserverDemo {
    // Entry point: register observers, push weather, then unregister one observer
    public static void main(String[] args) {
        WeatherStation station = new WeatherStation();
        Observer phone = new PhoneDisplay("Phone");
        Observer web = new PhoneDisplay("Web");

        station.register(phone);
        station.register(web);
        station.setWeather("Sunny, 26°C");

        station.unregister(web);
        station.setWeather("Light rain, 18°C");
    }
}
```

---

[Back to skill overview](../SKILL.md)
