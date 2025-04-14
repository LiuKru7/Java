# 📣 Spring Events Mini Example

This mini example demonstrates how to create and handle custom events in Spring. It helps decouple business logic using Spring's event mechanism.

---

## 1️⃣ Define the Custom Event

```java
public class UserRegisteredEvent extends ApplicationEvent {

    private final String email;

    public UserRegisteredEvent(Object source, String email) {
        super(source);
        this.email = email;
    }

    public String getEmail() {
        return email;
    }
}
```

---

## 2️⃣ Publish the Event

```java
@Service
@RequiredArgsConstructor
public class UserService {

    private final ApplicationEventPublisher eventPublisher;

    public void registerUser(String email) {
        // Business logic like saving to DB would be here
        System.out.println("User registered with email: " + email);

        // Publish custom event
        eventPublisher.publishEvent(new UserRegisteredEvent(this, email));
    }
}
```

---

## 3️⃣ Listen to the Event

```java
@Component
public class WelcomeEmailListener {

    @EventListener
    public void onUserRegistered(UserRegisteredEvent event) {
        // React to the event (e.g., send welcome email)
        System.out.println("Sending welcome email to: " + event.getEmail());
    }
}
```

---

## ✅ Output Example

```
User registered with email: user@example.com
Sending welcome email to: user@example.com
```

This shows how events can help split responsibilities (e.g., registration and email sending).