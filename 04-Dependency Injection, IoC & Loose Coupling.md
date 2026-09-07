# Dependency Injection, IoC & Loose Coupling


> **Core Theme:** Learn how to design Java applications that are loosely coupled, maintainable, testable, and easy to extend.

---

# 1. What Will We Learn?

This lecture establishes the foundation for understanding Spring's most important feature:

- Tight Coupling
- Loose Coupling
- Interfaces and Abstraction
- Dependency Injection (DI)
- Constructor Injection
- Setter Injection
- Field Injection
- Unit Testing with DI
- Inversion of Control (IoC)
- Dependency Inversion Principle
- Spring IoC Container
- Spring Beans
- Manual Object Wiring vs Spring-managed Wiring

---

# 2. The Big Picture

Before learning Spring, understand the problem Spring is trying to solve.

```text
                SOFTWARE DESIGN PROBLEM

        Tight Coupling
              │
              ▼
     Difficult to change
              │
              ▼
     Difficult to test
              │
              ▼
     Difficult to maintain
              │
              ▼
     Difficult to scale
              │
              ▼
      ┌─────────────────┐
      │     SOLUTION    │
      │                 │
      │  Abstraction    │
      │       +         │
      │ Dependency      │
      │   Injection     │
      │       +         │
      │      IoC        │
      └─────────────────┘
              │
              ▼
       Loose Coupling
````

---

# 3\. What is Coupling?

**Coupling** describes how strongly one class depends on another class.

For example:

```
class OrderService {

    private EmailService emailService;

}
```

`OrderService` depends on `EmailService`.

Therefore:

```
OrderService
     │
     │ depends on
     ▼
EmailService
```

The stronger the dependency between classes, the more tightly coupled they are.

---

# 4\. Tight Coupling

## Definition

**Tight coupling** occurs when a class is directly dependent on a specific concrete implementation.

Example:

```
public class OrderService {

    public void placeOrder() {

        System.out.println("Order Placed");

        EmailService email = new EmailService();

        email.sendNotification();
    }
}
```

Here:

```
EmailService email = new EmailService();
```

is the problem.

`OrderService` is:

1. Knowing the concrete class.
2. Creating the object.
3. Managing the dependency.
4. Using the dependency.

---

# 5\. Why Tight Coupling Is a Problem

Suppose the business initially uses email.

```
OrderService
     │
     ▼
EmailService
```

Later the business wants SMS.

```
OrderService
     │
     ▼
SmsService
```

We have to modify `OrderService`.

Later:

```
Email
SMS
WhatsApp
Push Notification
Slack
```

Every new implementation can force changes in `OrderService`.

This creates maintenance problems.

---

# 6\. Real-World Analogy — Delhi to Chandigarh

Imagine you need to travel from Delhi to Chandigarh.

## Tightly Coupled

You say:

> "I must travel using Bus #422, operated by Company X, at 9 AM, driven by Ramesh."

If:

- Bus #422 breaks
- Ramesh is unavailable
- Company X cancels the trip

your entire plan fails.

```
You
 │
 └──> Specific Bus
          │
          └──> Specific Driver
```

You are tightly coupled to implementation details.

---

## Loosely Coupled

Instead, you say:

> "I need transportation from Delhi to Chandigarh."

Now you can use:

```
Bus
Train
Taxi
Cab
Flight
```

```
                 Transportation
                       ▲
                       │
          ┌────────────┼────────────┐
          │            │            │
        Bus          Train         Taxi
```

You depend on the **abstraction/requirement**, not a specific implementation.

---

# 7\. Loose Coupling

Loose coupling means classes depend on abstractions rather than concrete implementations.

Instead of:

```
EmailService email = new EmailService();
```

we want:

```
NotificationService notification;
```

where:

```
interface NotificationService {
    void sendNotification();
}
```

Implementations:

```
                 NotificationService
                         ▲
            ┌────────────┼────────────┐
            │            │            │
            │            │            │
       EmailService  SmsService   PushService
```

Now `OrderService` does not need to know which implementation is being used.

---

# 8\. The Notification Example

Our application needs to send notifications after an order is placed.

Possible notification mechanisms:

```
                    Notification
                         │
          ┌──────────────┼──────────────┐
          ▼              ▼              ▼
        Email            SMS           Push
```

Instead of making `OrderService` depend on `EmailService`, create an abstraction.

---

# 9\. NotificationService Interface

```
package in.coderarmy.notification;

public interface NotificationService {

    void sendNotification();

}
```

This interface defines **what** a notification service should do.

It does not specify **how** it should do it.

---

# 10\. EmailService

```
public class EmailService implements NotificationService {

    @Override
    public void sendNotification() {
        System.out.println("Email notification sent");
    }
}
```

---

# 11\. SmsService

```
public class SmsService implements NotificationService {

    @Override
    public void sendNotification() {
        System.out.println("SMS notification sent");
    }
}
```

---

# 12\. PushNotificationService

```
public class PushNotificationService implements NotificationService {

    @Override
    public void sendNotification() {
        System.out.println("Push notification sent");
    }
}
```

---

# 13\. Stage 1 — Coding to an Interface

We might write:

```
NotificationService notification = new EmailService();
```

instead of:

```
EmailService notification = new EmailService();
```

This is better because the variable depends on the abstraction.

```
              NotificationService
                     ▲
                     │
                EmailService
```

However, there is still a problem.

---

# 14\. The Remaining Problem

Look carefully:

```
NotificationService notification = new EmailService();
```

Although the reference type is an interface, we are still creating the concrete object inside our class.

Example:

```
public class OrderService {

    public void placeOrder() {

        System.out.println("Order Placed");

        NotificationService notification =
                new EmailService();

        notification.sendNotification();
    }
}
```

`OrderService` still knows about:

```
new EmailService()
```

Therefore, coupling has been reduced but not completely removed.

---

# 15\. Dependency

A **dependency** is something a class needs in order to perform its job.

For example:

```
OrderService
     │
     │ needs
     ▼
NotificationService
```

Therefore:

> `NotificationService` is a dependency of `OrderService`.

---

# 16\. The Golden Rule of Dependency Injection

> **A class should ask for what it needs instead of creating what it needs.**

Bad:

```
class OrderService {

    public void placeOrder() {

        EmailService email = new EmailService();

    }
}
```

Better:

```
class OrderService {

    private NotificationService notification;

}
```

The dependency is supplied from outside.

That process is called:

# Dependency Injection

---

# 17\. What Is Dependency Injection?

**Dependency Injection (DI)** is a technique where an object's dependencies are supplied from outside instead of being created by the object itself.

Without DI:

```
OrderService
     │
     └── creates
           │
           ▼
     EmailService
```

With DI:

```
                 Main / Container
                  │          │
                  │ creates  │ creates
                  ▼          ▼
             EmailService  OrderService
                  │
                  │ injected into
                  ▼
             OrderService
```

---

# 18\. Three Common Types of Dependency Injection

There are three commonly discussed forms:

1. Constructor Injection
2. Setter Injection
3. Field Injection

---

# 19\. Constructor Injection

Constructor Injection means passing the dependency through the constructor.

```
public class OrderService {

    private final NotificationService notification;

    public OrderService(NotificationService notification) {
        this.notification = notification;
    }

    public void placeOrder() {

        System.out.println("Order Placed");

        notification.sendNotification();
    }
}
```

---

# 20\. Main.java with Constructor Injection

```
public class Main {

    public static void main(String[] args) {

        NotificationService notification =
                new EmailService();

        OrderService order =
                new OrderService(notification);

        order.placeOrder();
    }
}
```

Flow:

```
                 Main
                  │
          creates EmailService
                  │
                  ▼
          EmailService object
                  │
                  │ inject
                  ▼
            OrderService
                  │
                  ▼
             placeOrder()
                  │
                  ▼
       notification.sendNotification()
```

---

# 21\. Why Constructor Injection Is Preferred

Constructor injection is generally the preferred approach in Spring applications.

### Advantages

- Dependency is explicit.
- Required dependencies can be enforced.
- Object cannot easily exist in an invalid state.
- Makes classes easier to test.
- Works without Spring.
- Allows fields to be `final`.
- Makes dependencies visible immediately.

Example:

```
public class OrderService {

    private final NotificationService notification;

    public OrderService(NotificationService notification) {
        this.notification = notification;
    }
}
```

A developer immediately knows:

> OrderService requires NotificationService.

---

# 22\. Setter Injection

Setter Injection supplies the dependency through a setter method.

```
public class OrderService {

    private NotificationService notification;

    public void setNotification(
            NotificationService notification) {

        this.notification = notification;
    }

    public void placeOrder() {

        System.out.println("Order Placed");

        notification.sendNotification();
    }
}
```

Usage:

```
public class Main {

    public static void main(String[] args) {

        OrderService order = new OrderService();

        order.setNotification(
                new EmailService()
        );

        order.placeOrder();
    }
}
```

Flow:

```
Main
 │
 ├── new OrderService()
 │
 └── setNotification(EmailService)
              │
              ▼
        OrderService
```

---

# 23\. When Is Setter Injection Useful?

Setter injection can be useful when:

- A dependency is optional.
- A dependency may be changed after construction.
- You need to configure an object after creation.

However, for mandatory dependencies, constructor injection is generally preferred.

---

# 24\. Field Injection

Field Injection uses an annotation to inject the dependency directly into a field.

Example in Spring:

```
@Autowired
private NotificationService notification;
```

Conceptually:

```
Spring Container
      │
      │ injects
      ▼
private NotificationService notification;
```

Field injection relies on a framework/container to perform the injection.

### Important Interview Point

Field injection is generally discouraged in modern Spring applications because:

- Dependencies are hidden.
- Fields cannot naturally be `final`.
- Unit testing becomes less straightforward.
- The class can appear constructible without its required dependencies.

Prefer:

```
public OrderService(NotificationService notification)
```

over:

```
@Autowired
private NotificationService notification;
```

for mandatory dependencies.

---

# 25\. DI Comparison

| Type | Dependency Provided Through | Recommended? |
| --- | --- | --- |
| Constructor Injection | Constructor | ⭐ Yes |
| Setter Injection | Setter method | Good for optional/changeable dependencies |
| Field Injection | Field/annotation | Generally discouraged |

---

# 26\. Dependency Injection Does NOT Mean Spring

This is extremely important.

DI is **not a Spring-only concept**.

We can perform DI using pure Java.

Example:

```
NotificationService notification =
        new EmailService();

OrderService order =
        new OrderService(notification);
```

No Spring is involved.

Therefore:

```
Dependency Injection
        │
        ├── Can be done with Java
        │
        └── Can be automated by Spring
```

Spring makes dependency injection easier to manage at large scale.

---

# 27\. DI and Testability

One of the biggest advantages of DI is:

# Easier Unit Testing

Suppose `OrderService` directly creates:

```
new EmailService();
```

Testing becomes difficult because every test may trigger real email functionality.

---

# 28\. The Problem Without DI

```
public class OrderService {

    public void placeOrder() {

        EmailService email =
                new EmailService();

        email.sendNotification();
    }
}
```

During testing:

```
Test
 │
 ▼
OrderService
 │
 ▼
EmailService
 │
 ▼
Real Email System
 │
 ▼
Network
```

This is undesirable for a unit test.

---

# 29\. Fake Implementation

Create a fake implementation:

```
public class FakeNotificationService
        implements NotificationService {

    @Override
    public void sendNotification() {
        System.out.println("Fake notification");
    }
}
```

Now inject it:

```
NotificationService fake =
        new FakeNotificationService();

OrderService order =
        new OrderService(fake);

order.placeOrder();
```

Flow:

```
              OrderService
                   │
                   │ depends on
                   ▼
          NotificationService
                   ▲
                   │
        ┌──────────┴──────────┐
        │                     │
   EmailService        FakeNotificationService
   Production              Testing
```

The same `OrderService` can work with both.

---

# 30\. Mock vs Fake

These terms are often confused.

### Fake

A lightweight working implementation used for testing.

```
class FakeNotificationService
        implements NotificationService {

    public void sendNotification() {
        System.out.println("Fake notification");
    }
}
```

### Mock

A test double typically created using a mocking framework such as Mockito to verify interactions.

Example conceptually:

```
Did OrderService call
sendNotification() exactly once?
```

Interview point:

> DI makes it easy to replace real implementations with test doubles such as fakes or mocks.

---

# 31\. Loose Coupling — Final Architecture

After applying abstraction + DI:

```
                 OrderService
                       │
                       │ depends on
                       ▼
              NotificationService
                       ▲
             ┌─────────┼─────────┐
             │         │         │
             ▼         ▼         ▼
           Email      SMS       Push
```

`OrderService` doesn't care which implementation is used.

---

# 32\. Open-Closed Principle

One reason this design is better is the **Open-Closed Principle (OCP)**.

> Software entities should be open for extension but closed for modification.

Suppose we add:

```
class WhatsAppService
        implements NotificationService {
}
```

We don't need to modify:

```
OrderService
```

We simply provide the new implementation.

```
NotificationService
        ▲
        │
 ┌──────┼─────────────┐
 │      │             │
Email   SMS         WhatsApp
```

---

# 33\. Single Responsibility Principle

The original `OrderService` was doing two jobs:

```
OrderService
 │
 ├── Place order
 │
 └── Create notification object
```

After DI:

```
OrderService
 │
 └── Place order

Main / IoC Container
 │
 └── Create + wire objects
```

Now object construction is separated from business logic.

This improves adherence to **Single Responsibility Principle (SRP)**.

---

# 34\. Dependency Inversion Principle vs Dependency Injection

These are NOT the same thing.

This is an important interview distinction.

## Dependency Inversion Principle (DIP)

DIP is a **SOLID design principle**.

High-level modules should not depend directly on low-level concrete modules.

Both should depend on abstractions.

```
Bad:

OrderService
     │
     ▼
EmailService
```

Better:

```
        OrderService
             │
             ▼
    NotificationService
             ▲
             │
        EmailService
```

---

# 35\. Dependency Injection vs Dependency Inversion

| Concept | Meaning |
| --- | --- |
| Dependency Inversion Principle | Design principle |
| Dependency Injection | Technique for supplying dependencies |
| IoC | Broader principle of transferring control |
| Spring IoC Container | Framework implementation that manages objects/dependencies |

Remember:

```
DIP = Principle

DI = Technique

IoC = Broad architectural principle

Spring Container = Framework mechanism
```

---

# 36\. What Is Inversion of Control?

**Inversion of Control (IoC)** means transferring control of some part of the program from your code to an external component/framework.

Traditional Java:

```
Your Code
   │
   ├── creates objects
   ├── calls methods
   ├── controls flow
   └── manages lifecycle
```

With a framework:

```
Framework
   │
   ├── creates objects
   ├── manages objects
   ├── calls your code
   └── manages lifecycle
```

The control has been inverted.

---

# 37\. Simple IoC Example

Traditional Java:

```
public static void main(String[] args) {

    OrderService order =
            new OrderService();

    order.placeOrder();
}
```

You control:

```
Object creation
       ↓
Method invocation
       ↓
Program flow
```

With a framework, the framework can decide:

```
When to create OrderService
        ↓
Which dependency to provide
        ↓
When to invoke certain methods
        ↓
When to destroy/manage objects
```

That is IoC.

---

# 38\. IoC and DI Relationship

Think of IoC as the **big idea** and DI as one common technique used to achieve it.

```
                    IoC
                     │
        ┌────────────┴────────────┐
        │                         │
  Broad Principle          Control is transferred
        │
        ▼
       DI
        │
        ▼
 Dependencies supplied externally
```

---

# 39\. Before IoC / DI

```
Main
 │
 ▼
OrderService
 │
 │ creates
 ▼
EmailService
```

`OrderService` controls the creation of its dependency.

---

# 40\. After DI

```
             Main
              │
       creates objects
              │
       ┌──────┴──────┐
       ▼             ▼
 EmailService    OrderService
       │             ▲
       │             │
       └── injected ─┘
```

`OrderService` no longer creates `EmailService`.

---

# 41\. The Manual Wiring Problem

DI works beautifully for small applications.

But imagine an application containing:

```
Database
UserRepository
OrderRepository
PaymentGateway
EmailService
UserService
OrderService
PaymentService
AuthenticationService
NotificationService
...
```

Manually creating everything becomes painful.

Example:

```
Database db = new Database();

UserRepository userRepo =
        new UserRepository(db);

EmailService email =
        new EmailService();

UserService userService =
        new UserService(userRepo, email);

PaymentGateway gateway =
        new PaymentGateway();

OrderRepository orderRepo =
        new OrderRepository(db);

OrderService orderService =
        new OrderService(
                orderRepo,
                userService,
                gateway
        );
```

Imagine doing this for 500 classes.

This is where Spring becomes extremely useful.

---

# 42\. Spring IoC Container

Spring provides an **IoC Container**.

The container is responsible for:

1. Creating objects.
2. Managing objects.
3. Resolving dependencies.
4. Injecting dependencies.
5. Managing object lifecycle.

```
                    Spring IoC Container
                            │
            ┌───────────────┼───────────────┐
            │               │               │
            ▼               ▼               ▼
       UserService     OrderService    EmailService
            │               │               │
            └───────────────┼───────────────┘
                            │
                      Dependency Wiring
```

---

# 43\. What Is a Spring Bean?

A **Spring Bean** is an object that is instantiated and managed by the Spring IoC container.

Important:

> Every Spring Bean is a Java object, but not every Java object is a Spring Bean.

Example:

```
EmailService email =
        new EmailService();
```

This is simply a Java object.

But if Spring creates and manages it:

```
Spring Container
       │
       ▼
EmailService object
       │
       ▼
Spring Bean
```

it becomes a Spring Bean.

---

# 44\. Bean Definition

A bean is essentially an object whose lifecycle and configuration are managed by Spring.

Common ways to register beans include:

```
@Component
@Service
@Repository
@Controller
```

or explicitly:

```
@Bean
```

Example:

```
@Service
public class EmailService {
}
```

Spring can detect this class through component scanning and create a bean for it.

---

# 45\. Spring Dependency Injection Example

```
@Service
public class EmailService {

    public void sendNotification() {
        System.out.println("Email sent");
    }
}
```

```
@Service
public class OrderService {

    private final EmailService emailService;

    public OrderService(EmailService emailService) {
        this.emailService = emailService;
    }

    public void placeOrder() {
        System.out.println("Order Placed");
        emailService.sendNotification();
    }
}
```

Spring can:

```
1. Find EmailService
        ↓
2. Create EmailService bean
        ↓
3. Find OrderService
        ↓
4. See constructor dependency
        ↓
5. Inject EmailService
        ↓
6. Create OrderService bean
```

---

# 46\. Interface-Based Injection in Spring

For better flexibility:

```
public interface NotificationService {

    void sendNotification();

}
```

Implementation:

```
@Service
public class EmailService
        implements NotificationService {

    @Override
    public void sendNotification() {
        System.out.println("Email sent");
    }
}
```

Consumer:

```
@Service
public class OrderService {

    private final NotificationService notificationService;

    public OrderService(
            NotificationService notificationService) {

        this.notificationService =
                notificationService;
    }
}
```

Now `OrderService` depends on:

```
NotificationService
```

rather than:

```
EmailService
```

---

# 47\. What Happens If Multiple Implementations Exist?

Suppose:

```
@Service
class EmailService implements NotificationService {
}
```

and:

```
@Service
class SmsService implements NotificationService {
}
```

Now Spring sees:

```
NotificationService
       ▲
       │
 ┌─────┴─────┐
 │           │
Email       SMS
```

If Spring doesn't know which one should be injected, you can get an ambiguity error.

Solutions include:

```
@Primary
```

or:

```
@Qualifier("emailService")
```

Example:

```
public OrderService(
    @Qualifier("emailService")
    NotificationService notificationService) {

    this.notificationService =
            notificationService;
}
```

This is a very common Spring interview topic.

---

# 48\. Manual DI vs Spring DI

## Without Spring

```
NotificationService notification =
        new EmailService();

OrderService order =
        new OrderService(notification);
```

You manually wire everything.

## With Spring

```
@Service
class OrderService {

    private final NotificationService notification;

    public OrderService(
            NotificationService notification) {

        this.notification = notification;
    }
}
```

Spring performs the wiring.

```
Manual Java
    │
    ▼
Developer wires dependencies

Spring
    │
    ▼
IoC Container wires dependencies
```

---

# 49\. Does Spring Eliminate `new` Completely?

A common classroom statement is:

> "In Spring, you never use `new`."

This is an oversimplification.

More accurate:

> **For objects managed by Spring, you generally don't manually instantiate those application components with `new`; Spring creates and manages them.**

You can still legitimately use `new` in Spring applications for:

- DTOs
- value objects
- utility objects
- collections
- objects intentionally created outside the container

Example:

```
UserDto dto = new UserDto();
```

is perfectly normal.

---

# 50\. Complete Architecture

```
                     CLIENT
                       │
                       ▼
                 Spring Application
                       │
                       ▼
               ┌───────────────┐
               │ IoC Container │
               └───────┬───────┘
                       │
          ┌────────────┼────────────┐
          │            │            │
          ▼            ▼            ▼
     Controller     Service     Repository
                       │
                       ▼
              NotificationService
                       ▲
              ┌────────┴────────┐
              │                 │
              ▼                 ▼
        EmailService        SmsService
```

---

# 51\. Important Terminology

| Term | Meaning |
| --- | --- |
| Dependency | Object/class another class needs |
| Coupling | Degree of dependency between components |
| Tight Coupling | Strong dependency on concrete implementations |
| Loose Coupling | Dependency on abstractions |
| Interface | Defines a contract |
| DI | Supplying dependencies from outside |
| Constructor Injection | Dependency passed through constructor |
| Setter Injection | Dependency supplied through setter |
| Field Injection | Dependency injected directly into a field |
| IoC | Transfer of control to an external framework/container |
| IoC Container | Creates and manages Spring objects |
| Bean | Object managed by Spring |
| DIP | SOLID principle favoring abstractions |
| OCP | Open for extension, closed for modification |
| SRP | A class should have a single responsibility |

---

# 52\. Interview Question Bank

## Beginner Level

### Q1. What is coupling?

**Answer:**

Coupling represents the degree of dependency between classes or modules.

High coupling means classes depend heavily on each other, while low/loose coupling means components interact through abstractions with fewer implementation details exposed.

---

### Q2. What is tight coupling?

**Answer:**

Tight coupling occurs when a class directly depends on a specific concrete implementation.

Example:

```
EmailService email =
        new EmailService();
```

inside `OrderService`.

---

### Q3. What is loose coupling?

**Answer:**

Loose coupling means minimizing direct dependencies between classes, usually by depending on abstractions such as interfaces.

Example:

```
private NotificationService notification;
```

instead of:

```
private EmailService email;
```

---

### Q4. What is a dependency?

**Answer:**

A dependency is an object or component that another class requires to perform its work.

For example:

```
OrderService
      │
      ▼
NotificationService
```

`NotificationService` is a dependency of `OrderService`.

---

### Q5. What is Dependency Injection?

**Answer:**

Dependency Injection is a technique in which dependencies are supplied to a class from outside rather than being created by the class itself.

---

### Q6. Is Dependency Injection a Spring concept?

**Answer:**

No.

DI is a general software design technique and can be implemented using plain Java.

Spring provides an IoC container that automates dependency creation and injection.

---

# 53\. Intermediate Interview Questions

### Q7. What are the types of Dependency Injection?

**Answer:**

Three commonly discussed types are:

1. Constructor Injection
2. Setter Injection
3. Field Injection

Constructor injection is generally preferred for mandatory dependencies.

---

### Q8. Why is constructor injection preferred?

**Answer:**

Because it:

- Makes dependencies explicit.
- Supports immutable fields.
- Ensures required dependencies are provided during construction.
- Makes unit testing easier.
- Reduces hidden framework magic.
- Helps prevent partially initialized objects.

---

### Q9. What is Setter Injection?

**Answer:**

Setter Injection supplies a dependency through a setter method after object construction.

```
public void setNotificationService(
        NotificationService service) {

    this.service = service;
}
```

It can be useful for optional or changeable dependencies.

---

### Q10. What is Field Injection?

**Answer:**

Field Injection injects a dependency directly into a field, typically using an annotation such as:

```
@Autowired
private NotificationService service;
```

It is generally less preferred than constructor injection because dependencies are less explicit and testing can be more cumbersome.

---

### Q11. What is IoC?

**Answer:**

IoC, or Inversion of Control, is a principle in which control over some part of object creation, dependency management, lifecycle, or program execution is transferred from application code to an external framework or container.

---

### Q12. Difference between IoC and DI?

**Answer:**

IoC is the broader principle of transferring control.

DI is a technique for providing dependencies externally.

```
IoC
 │
 └── DI is one way to implement IoC
```

---

# 54\. Advanced Interview Questions

### Q13. What is Dependency Inversion Principle?

**Answer:**

DIP is the fifth SOLID principle.

It states that high-level modules should not depend directly on low-level concrete implementations. Both should depend on abstractions.

Example:

```
Bad:

OrderService
     │
     ▼
EmailService
```

Better:

```
OrderService
     │
     ▼
NotificationService
     ▲
     │
EmailService
```

---

### Q14. Is Dependency Inversion the same as Dependency Injection?

**Answer:**

No.

Dependency Inversion is a **design principle**.

Dependency Injection is a **technique** for supplying dependencies.

They are related but not identical.

---

### Q15. How does DI improve testing?

**Answer:**

DI allows us to replace real dependencies with test doubles.

Production:

```
OrderService
     │
     ▼
EmailService
```

Testing:

```
OrderService
     │
     ▼
FakeNotificationService
```

Therefore tests don't need to perform real network calls or external operations.

---

### Q16. What is the role of the Spring IoC Container?

**Answer:**

The Spring IoC Container:

- Creates beans.
- Configures beans.
- Resolves dependencies.
- Injects dependencies.
- Manages bean lifecycle.
- Maintains the objects it manages.

---

### Q17. What is a Spring Bean?

**Answer:**

A Spring Bean is an object that is instantiated and managed by the Spring IoC container.

---

### Q18. Is every Java object a Spring Bean?

**Answer:**

No.

Only objects managed by the Spring IoC container are Spring Beans.

```
Java Object
    │
    ├── Managed by Spring → Spring Bean
    │
    └── Not managed by Spring → Regular Java Object
```

---

### Q19. What happens when Spring sees constructor injection?

Suppose:

```
@Service
class OrderService {

    private final NotificationService notification;

    public OrderService(
            NotificationService notification) {
        this.notification = notification;
    }
}
```

Spring:

```
Find OrderService
       ↓
Inspect constructor
       ↓
Find NotificationService dependency
       ↓
Find suitable bean
       ↓
Create/inject dependency
       ↓
Create OrderService
```

---

### Q20. What happens if two beans implement the same interface?

Suppose:

```
EmailService implements NotificationService
SmsService implements NotificationService
```

Spring may not know which one to inject.

Solutions:

```
@Primary
```

or:

```
@Qualifier
```

---

# 55\. Tricky Interview Questions

## Q21. If I use an interface, am I automatically loosely coupled?

**Answer:**

Not necessarily.

Consider:

```
NotificationService notification =
        new EmailService();
```

If this concrete creation happens inside the consumer class, the class still knows about `EmailService`.

Using an interface helps with abstraction, but dependency creation should also be separated.

---

## Q22. Can DI exist without interfaces?

**Answer:**

Yes.

You can inject a concrete class:

```
public OrderService(
        EmailService emailService) {
}
```

This is still Dependency Injection.

However, interfaces can provide greater flexibility and abstraction when multiple implementations are expected.

---

## Q23. Can we perform DI without Spring?

**Answer:**

Yes.

```
NotificationService service =
        new EmailService();

OrderService order =
        new OrderService(service);
```

This is manual Dependency Injection.

---

## Q24. Why do we need Spring if DI can be done manually?

**Answer:**

Manual DI becomes difficult as the application grows.

With hundreds of classes and deeply nested dependencies, manually creating and wiring objects becomes complex.

Spring's IoC container automates:

```
Object creation
       +
Dependency resolution
       +
Dependency injection
       +
Lifecycle management
```

---

## Q25. What is the main problem Spring solves here?

**Answer:**

Spring helps manage object creation and dependencies while promoting loosely coupled architecture.

Instead of classes creating their own dependencies:

```
new EmailService()
```

the container can create and inject the required object.

---

# 56\. Code Comparison — Before vs After

## ❌ Bad: Tight Coupling

```
public class OrderService {

    public void placeOrder() {

        System.out.println("Order Placed");

        EmailService email =
                new EmailService();

        email.sendNotification();
    }
}
```

Problems:

```
❌ Concrete dependency
❌ Object creation inside business class
❌ Difficult to replace
❌ Difficult to test
❌ Violates DIP
```

---

## ⚠️ Better: Interface but Manual Creation

```
public class OrderService {

    public void placeOrder() {

        NotificationService notification =
                new EmailService();

        notification.sendNotification();
    }
}
```

Better abstraction, but:

```
❌ Concrete creation still inside OrderService
```

---

## ✅ Better: Constructor DI

```
public class OrderService {

    private final NotificationService notification;

    public OrderService(
            NotificationService notification) {

        this.notification = notification;
    }

    public void placeOrder() {

        System.out.println("Order Placed");

        notification.sendNotification();
    }
}
```

Now:

```
OrderService
      │
      ▼
NotificationService
```

---

# 57\. Complete Pure Java Example

## NotificationService.java

```
public interface NotificationService {

    void sendNotification();

}
```

## EmailService.java

```
public class EmailService
        implements NotificationService {

    @Override
    public void sendNotification() {
        System.out.println(
            "Email notification sent"
        );
    }
}
```

## SmsService.java

```
public class SmsService
        implements NotificationService {

    @Override
    public void sendNotification() {
        System.out.println(
            "SMS notification sent"
        );
    }
}
```

## OrderService.java

```
public class OrderService {

    private final NotificationService notification;

    public OrderService(
            NotificationService notification) {

        this.notification = notification;
    }

    public void placeOrder() {

        System.out.println("Order Placed");

        notification.sendNotification();
    }
}
```

## Main.java

```
public class Main {

    public static void main(String[] args) {

        NotificationService notification =
                new EmailService();

        OrderService order =
                new OrderService(notification);

        order.placeOrder();
    }
}
```

Output:

```
Order Placed
Email notification sent
```

---

# 58\. Switching Implementations

To switch from Email to SMS:

```
NotificationService notification =
        new SmsService();
```

Everything else remains unchanged.

```
Before:

Main
 │
 ▼
EmailService
 │
 ▼
OrderService

After:

Main
 │
 ▼
SmsService
 │
 ▼
OrderService
```

`OrderService` remains unchanged.

That is the power of loose coupling.

---

# 59\. Testing Example

```
public class FakeNotificationService
        implements NotificationService {

    @Override
    public void sendNotification() {
        System.out.println(
            "Fake notification sent"
        );
    }
}
```

Test:

```
public class Main {

    public static void main(String[] args) {

        NotificationService fake =
                new FakeNotificationService();

        OrderService order =
                new OrderService(fake);

        order.placeOrder();
    }
}
```

Output:

```
Order Placed
Fake notification sent
```

No real email/SMS system is required.

---

# 60\. Dependency Graph

A useful way to visualize an application is as a dependency graph.

```
                    OrderService
                         │
                         ▼
               NotificationService
                         ▲
              ┌──────────┴──────────┐
              │                     │
              ▼                     ▼
        EmailService           SmsService
```

The key architectural rule:

```
Consumer
   │
   ▼
Abstraction
   ▲
   │
Implementation
```

---

# 61\. The Evolution of the Code

```
LEVEL 1
Tight Coupling

OrderService
     │
     ▼
EmailService
```

↓

```
LEVEL 2
Interface

OrderService
     │
     ▼
NotificationService
     ▲
     │
EmailService
```

↓

```
LEVEL 3
Dependency Injection

Main
 │
 ├── creates EmailService
 │
 └── injects it
        │
        ▼
   OrderService
```

↓

```
LEVEL 4
Spring IoC

Spring Container
       │
       ├── creates EmailService
       │
       ├── creates OrderService
       │
       └── injects EmailService
                 │
                 ▼
            OrderService
```

---

# 62\. Most Important Interview Diagram

Memorize this:

```
                 SPRING IoC CONTAINER
                         │
            ┌────────────┴────────────┐
            │                         │
       Creates Bean              Creates Bean
            │                         │
            ▼                         ▼
     EmailService              OrderService
                                      │
                                      │ requires
                                      ▼
                             NotificationService
                                      ▲
                                      │
                               EmailService
```

Spring acts as the **object factory + dependency manager + lifecycle manager**.

---

# 63\. One-Minute Interview Explanation

If the interviewer asks:

> "Explain Dependency Injection and IoC."

A strong answer:

> Dependency Injection is a design technique where a class receives its dependencies from outside instead of creating them itself. For example, instead of `OrderService` creating an `EmailService` using `new`, we can pass a `NotificationService` through the constructor.
>
>  This reduces coupling, improves testability, and makes implementations easier to replace.
>
>  IoC, or Inversion of Control, is the broader principle where control over object creation, dependency management, or lifecycle is transferred from application code to an external component. In Spring, the IoC container manages these responsibilities and injects dependencies into Spring Beans.

---

# 64\. Rapid Revision

```
Tight Coupling
    ↓
Concrete dependency
    ↓
Hard to change
    ↓
Hard to test
    ↓
Introduce abstraction
    ↓
Interface
    ↓
Dependency Injection
    ↓
Dependency supplied externally
    ↓
Loose Coupling
    ↓
Spring IoC Container
    ↓
Automatic object creation + wiring
    ↓
Spring Beans
```

---

# 65\. Must-Remember Statements

### ⭐ Statement 1

> A class should ask for what it needs rather than creating it itself.

### ⭐ Statement 2

> Dependency Injection is a technique; IoC is the broader principle.

### ⭐ Statement 3

> DI can be implemented without Spring.

### ⭐ Statement 4

> Spring automates dependency management using its IoC container.

### ⭐ Statement 5

> Constructor injection is generally preferred for mandatory dependencies.

### ⭐ Statement 6

> Every Spring Bean is a Java object, but not every Java object is a Spring Bean.

### ⭐ Statement 7

> Programming to an interface reduces dependency on concrete implementations.

### ⭐ Statement 8

> Dependency Inversion Principle and Dependency Injection are related but not the same.

---

# 66\. Interview Cheat Sheet

| Question | Short Answer |
| --- | --- |
| What is coupling? | Degree of dependency between components |
| Tight coupling? | Strong dependency on concrete implementation |
| Loose coupling? | Dependence minimized through abstractions |
| Dependency? | Object required by another object |
| DI? | Supplying dependency from outside |
| DI without Spring? | Yes |
| Types of DI? | Constructor, Setter, Field |
| Preferred DI? | Constructor |
| IoC? | Transfer of control to external framework/container |
| DI vs IoC? | DI is a technique; IoC is broader principle |
| DIP? | SOLID principle favoring abstractions |
| Spring IoC Container? | Creates, wires and manages Spring Beans |
| Spring Bean? | Object managed by Spring container |
| Multiple implementations? | Use `@Primary` or `@Qualifier` |
| Main benefit of DI? | Loose coupling + testability |
| Why Spring? | Automates object creation, wiring and lifecycle management |

---

# 67\. Final Mental Model

Don't memorize Spring annotations first.

Understand this progression:

```
                    PROBLEM
                       │
                       ▼
               Tight Coupling
                       │
                       ▼
            Depend on concrete class
                       │
                       ▼
                Hard to change
                       │
                       ▼
               Hard to test
                       │
                       ▼
              Use Abstraction
                       │
                       ▼
                   Interface
                       │
                       ▼
             Dependency Injection
                       │
                       ▼
               Loose Coupling
                       │
                       ▼
          Manual wiring becomes large
                       │
                       ▼
                Spring IoC Container
                       │
                       ▼
             Automatic object creation
                       │
                       ▼
             Automatic dependency wiring
                       │
                       ▼
                 Spring Beans
                       │
                       ▼
             Maintainable application
```

# 🎯 Final Takeaway

The fundamental problem Spring solves is **not simply "how to create objects."**

The deeper problem is:

> **How do we manage the relationships and dependencies between hundreds or thousands of objects without tightly coupling them together?**

The progression is:

```
Tight Coupling
      ↓
Abstraction
      ↓
Loose Coupling
      ↓
Dependency Injection
      ↓
Inversion of Control
      ↓
Spring IoC Container
      ↓
Spring Beans
      ↓
Scalable & Maintainable Application
```

Once this mental model is clear, annotations such as:

```
@Component
@Service
@Repository
@Autowired
@Bean
@Primary
@Qualifier
```

become much easier to understand because you know **why they exist**, rather than merely memorizing them.
