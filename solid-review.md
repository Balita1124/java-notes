The **SOLID** principles are a set of design principles aimed at improving code readability, maintainability, and flexibility. Here’s a breakdown of each SOLID principle along with an example for each in Java, particularly in a **Jakarta EE** context:

### **1. Single Responsibility Principle (SRP)**
- **Definition**: A class should have only one reason to change, meaning it should have only one responsibility.
  
- **Violation**: A class does too many things (e.g., manages database operations, performs business logic, and handles security in one class).

- **Example (Bad)**:
```java
public class UserService {
    public void registerUser(User user) {
        // Validation logic
        if (user.getEmail() == null || user.getPassword() == null) {
            throw new IllegalArgumentException("Invalid user details");
        }
        
        // Persisting user to the database
        EntityManager em = getEntityManager();
        em.persist(user);

        // Sending welcome email
        sendWelcomeEmail(user);
    }

    private void sendWelcomeEmail(User user) {
        // Email sending logic
    }
}
```

- **Review & Refactor**: In this case, the `UserService` class is doing too much (validation, persistence, and sending emails). This violates SRP. 

- **Refactored Example (Good)**:
```java
public class UserService {
    private final UserRepository userRepository;
    private final EmailService emailService;
    private final UserValidator userValidator;

    public UserService(UserRepository userRepository, EmailService emailService, UserValidator userValidator) {
        this.userRepository = userRepository;
        this.emailService = emailService;
        this.userValidator = userValidator;
    }

    public void registerUser(User user) {
        userValidator.validate(user);
        userRepository.save(user);
        emailService.sendWelcomeEmail(user);
    }
}
```

Here, the responsibilities are separated into `UserValidator`, `UserRepository`, and `EmailService`.

---

### **2. Open/Closed Principle (OCP)**
- **Definition**: Software entities (classes, modules, functions) should be open for extension but closed for modification.
  
- **Violation**: Changing a class every time new functionality is added instead of extending the behavior.

- **Example (Bad)**:
```java
public class DiscountService {
    public double applyDiscount(Order order) {
        if (order.getType().equals("SENIOR")) {
            return order.getPrice() * 0.9; // 10% discount
        } else if (order.getType().equals("STUDENT")) {
            return order.getPrice() * 0.8; // 20% discount
        }
        return order.getPrice();
    }
}
```

- **Review & Refactor**: Every time a new type of discount is introduced (e.g., "Military"), you would have to modify the `DiscountService` class.

- **Refactored Example (Good)**:
```java
public interface DiscountStrategy {
    double applyDiscount(Order order);
}

public class SeniorDiscount implements DiscountStrategy {
    @Override
    public double applyDiscount(Order order) {
        return order.getPrice() * 0.9;
    }
}

public class StudentDiscount implements DiscountStrategy {
    @Override
    public double applyDiscount(Order order) {
        return order.getPrice() * 0.8;
    }
}

public class DiscountService {
    private final DiscountStrategy discountStrategy;

    public DiscountService(DiscountStrategy discountStrategy) {
        this.discountStrategy = discountStrategy;
    }

    public double applyDiscount(Order order) {
        return discountStrategy.applyDiscount(order);
    }
}
```

Here, `DiscountService` is open for extension (you can add new discount types), but closed for modification (the existing logic doesn't need to change).

---

### **3. Liskov Substitution Principle (LSP)**
- **Definition**: Subtypes must be substitutable for their base types without affecting the correctness of the program.
  
- **Violation**: Creating subclasses that break the expected behavior of the parent class.

- **Example (Bad)**:
```java
public class Rectangle {
    protected int width;
    protected int height;

    public void setWidth(int width) {
        this.width = width;
    }

    public void setHeight(int height) {
        this.height = height;
    }

    public int getArea() {
        return width * height;
    }
}

public class Square extends Rectangle {
    @Override
    public void setWidth(int width) {
        this.width = width;
        this.height = width; // Violation: modifying height unexpectedly
    }

    @Override
    public void setHeight(int height) {
        this.width = height;
        this.height = height; // Violation: modifying width unexpectedly
    }
}
```

- **Review & Refactor**: `Square` should not inherit from `Rectangle` since it changes the behavior of `setWidth` and `setHeight`. This breaks LSP.

- **Refactored Example (Good)**:
```java
public interface Shape {
    int getArea();
}

public class Rectangle implements Shape {
    private int width;
    private int height;

    public Rectangle(int width, int height) {
        this.width = width;
        this.height = height;
    }

    @Override
    public int getArea() {
        return width * height;
    }
}

public class Square implements Shape {
    private int side;

    public Square(int side) {
        this.side = side;
    }

    @Override
    public int getArea() {
        return side * side;
    }
}
```

Now, `Rectangle` and `Square` implement the same `Shape` interface and behave independently.

---

### **4. Interface Segregation Principle (ISP)**
- **Definition**: A client should not be forced to implement interfaces they do not use.

- **Violation**: Having an interface with too many methods, forcing classes to implement methods they don't need.

- **Example (Bad)**:
```java
public interface Worker {
    void work();
    void eat();
}

public class Developer implements Worker {
    @Override
    public void work() {
        // working
    }

    @Override
    public void eat() {
        // unnecessary: developers may not need to "eat" in the context of this app
    }
}
```

- **Review & Refactor**: The interface `Worker` contains methods that all implementations may not need.

- **Refactored Example (Good)**:
```java
public interface Workable {
    void work();
}

public interface Eatable {
    void eat();
}

public class Developer implements Workable {
    @Override
    public void work() {
        // working
    }
}

public class OfficeWorker implements Workable, Eatable {
    @Override
    public void work() {
        // working
    }

    @Override
    public void eat() {
        // eating
    }
}
```

Now, `Developer` only implements what it needs (`Workable`), and `OfficeWorker` can implement both `Workable` and `Eatable`.

---

### **5. Dependency Inversion Principle (DIP)**
- **Definition**: High-level modules should not depend on low-level modules. Both should depend on abstractions. Abstractions should not depend on details, details should depend on abstractions.

- **Violation**: A high-level class directly depends on a concrete implementation instead of an abstraction.

- **Example (Bad)**:
```java
public class EmailService {
    public void sendEmail(String message) {
        // send email
    }
}

public class UserService {
    private final EmailService emailService = new EmailService(); // tightly coupled

    public void registerUser(User user) {
        // register logic
        emailService.sendEmail("Welcome!");
    }
}
```

- **Review & Refactor**: `UserService` is tightly coupled to `EmailService`. If we want to change how we send messages (e.g., SMS), we would need to modify `UserService`.

- **Refactored Example (Good)**:
```java
public interface MessageService {
    void sendMessage(String message);
}

public class EmailService implements MessageService {
    @Override
    public void sendMessage(String message) {
        // send email
    }
}

public class UserService {
    private final MessageService messageService;

    public UserService(MessageService messageService) {
        this.messageService = messageService;
    }

    public void registerUser(User user) {
        // register logic
        messageService.sendMessage("Welcome!");
    }
}
```

Here, `UserService` depends on the `MessageService` abstraction. You can switch between `EmailService` or other implementations (e.g., SMS) without modifying `UserService`.

---

These principles help make your code easier to maintain, extend, and refactor over time. By adhering to SOLID, you reduce tight coupling, improve testability, and create a more flexible system.
