# Spring Core - IoC, Dependency Injection, Beans, Autowiring & Configuration

> **Level:** Beginner → Advanced\
>  **Focus:** Spring Core, IoC, Dependency Injection, Beans, Component Scanning, Autowiring, `@Primary`, `@Qualifier`, `@Bean`, Reflection, `BeanDefinition`, `ApplicationContext`, `BeanFactory`\
>  **Package used in examples:** `in.coderarmy`

---

# 1\. What Problem Does Spring Actually Solve?

Before understanding Spring, first understand the problem that Spring is trying to solve.

Consider an e-commerce application.

We have:

```
OrderService
     |
     | needs
     v
PaymentService
```

When an order is placed, the application needs to make a payment.

A naive implementation might look like this:

```
public class OrderService {

    private PaymentService paymentService = new PaymentService();

    public void placeOrder() {
        paymentService.pay();
        System.out.println("Order Placed");
    }
}
```

At first glance, this works perfectly.

But there is a design problem.

`OrderService` is responsible for:

1. Placing the order.
2. Deciding which payment implementation to use.
3. Creating the payment object.

That means `OrderService` is doing too much.

---

# 2\. Tight Coupling

## 2.1 What is tight coupling?

Two classes are tightly coupled when one class directly depends on the concrete implementation of another class.

For example:

```
public class OrderService {

    private PaymentService paymentService =
            new PaymentService();

}
```

Here:

```
OrderService
     |
     | directly creates
     v
PaymentService
```

`OrderService` knows:

> "I must create a `PaymentService` using `new`."

This creates strong coupling.

---

# 3\. Why Is Tight Coupling a Problem?

Imagine tomorrow we introduce:

```
CardPayment
UpiPayment
NetBankingPayment
WalletPayment
CryptoPayment
```

Our original code may become:

```
public class OrderService {

    private PaymentService paymentService;

    public OrderService() {
        paymentService = new PaymentService();
    }
}
```

If we want UPI:

```
paymentService = new UpiPayment();
```

If we want card:

```
paymentService = new CardPayment();
```

Now `OrderService` needs to know about concrete implementations.

That violates an important design principle:

> **Program to abstractions rather than concrete implementations.**

---

# 4\. Single Responsibility Principle

The **Single Responsibility Principle (SRP)** says that a class should have one primary responsibility.

A better separation is:

```
+----------------------+
|      Main / Config   |
|----------------------|
| Creates objects      |
| Connects dependencies|
+----------+-----------+
           |
           v
+----------------------+
|    OrderService      |
|----------------------|
| Places orders        |
+----------+-----------+
           |
           v
+----------------------+
|   PaymentService     |
|----------------------|
| Handles payment      |
+----------------------+
```

Now:

- `OrderService` → places orders.
- `PaymentService` → performs payment.
- Some external mechanism → creates and connects objects.

That external mechanism could initially be our `Main` class.

Later, Spring takes over that responsibility.

---

# 5\. Dependency Injection

The object that another object needs is called a **dependency**.

For example:

```
OrderService
```

needs:

```
PaymentService
```

Therefore:

```
PaymentService = Dependency
OrderService   = Dependent object
```

## Definition

> **Dependency Injection (DI) is a design technique in which an object's dependencies are supplied from outside rather than being created by the object itself.**

Instead of:

```
public class OrderService {

    private PaymentService paymentService =
            new PaymentService();

}
```

we write:

```
public class OrderService {

    private final PaymentService paymentService;

    public OrderService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }
}
```

Now `OrderService` does not create its dependency.

Someone else provides it.

---

# 6\. Manual Dependency Injection

Before Spring, we can perform DI manually.

## PaymentService

```
package in.coderarmy;

public class PaymentService {

    public void pay() {
        System.out.println("Payment Done");
    }
}
```

## OrderService

```
package in.coderarmy;

public class OrderService {

    private final PaymentService paymentService;

    public OrderService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }

    public void placeOrder() {

        paymentService.pay();

        System.out.println("Order Placed");
    }
}
```

## Main

```
package in.coderarmy;

public class Main {

    public static void main(String[] args) {

        // Create dependency
        PaymentService payment =
                new PaymentService();

        // Inject dependency
        OrderService order =
                new OrderService(payment);

        // Use object
        order.placeOrder();
    }
}
```

Output:

```
Payment Done
Order Placed
```

---

# 7\. What Did We Achieve?

Previously:

```
OrderService
     |
     | new PaymentService()
     v
PaymentService
```

Now:

```
              Main
               |
               | creates
               v
       PaymentService
               |
               | injects
               v
         OrderService
```

`OrderService` no longer controls dependency creation.

This is **Dependency Injection**.

---

# 8\. But What Is Still a Problem?

Imagine an application containing:

```
1000 classes
500 services
300 repositories
200 controllers
```

Manually writing:

```
A a = new A();

B b = new B(a);

C c = new C(b);

D d = new D(c);

E e = new E(d);
```

would become difficult to maintain.

Imagine a dependency graph like:

```
Application
    |
    +---- OrderService
    |        |
    |        +---- PaymentService
    |        |        |
    |        |        +---- PaymentGateway
    |        |
    |        +---- InventoryService
    |                 |
    |                 +---- ProductRepository
    |
    +---- UserService
             |
             +---- UserRepository
```

Manually creating all these objects becomes painful.

This is where Spring enters the picture.

---

# 9\. What Is Spring IoC?

Spring provides an **IoC Container**.

IoC means:

> **Inversion of Control**

Normally, our application controls object creation:

```
PaymentService payment =
        new PaymentService();
```

With Spring:

```
                Spring Container
                      |
          +-----------+-----------+
          |                       |
          v                       v
 PaymentService             OrderService
```

Spring controls:

- object creation
- dependency resolution
- dependency injection
- bean lifecycle
- configuration
- scopes
- initialization
- destruction

The control of object creation moves from our application code to the Spring container.

That is **Inversion of Control**.

---

# 10\. IoC vs DI

These two terms are related but not identical.

## IoC

IoC is the broader principle.

> Control over object creation and dependency management is transferred to an external container/framework.

## DI

DI is one way to implement IoC.

```
             Inversion of Control
                     |
                     v
          Dependency Injection
                     |
          +----------+----------+
          |          |          |
          v          v          v
     Constructor   Setter     Field
        DI           DI         DI
```

A good interview answer:

> **IoC is a design principle where control is inverted from the application to an external framework/container. Dependency Injection is a technique used to implement IoC by supplying dependencies from outside the dependent object.**

---

# 11\. Spring Core Dependency

For a simple console application, we do not necessarily need Spring Boot.

We can use Spring Framework directly.

A commonly used module is:

```
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>6.1.6</version>
</dependency>
```

The exact Spring version should be selected based on your project's Java version and the currently supported Spring release.

---

# 12\. What Is a Maven Dependency?

When we add:

```
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>...</version>
</dependency>
```

we are telling Maven:

> Download this library and make it available to my application.

Maven downloads dependencies into the local Maven repository.

Conceptually:

```
pom.xml
   |
   | declares
   v
spring-context
   |
   +---- spring-beans
   |
   +---- spring-core
   |
   +---- spring-aop
   |
   +---- spring-expression
   |
   +---- logging dependencies
```

---

# 13\. Transitive Dependencies

Suppose:

```
A depends on B
B depends on C
```

If our project depends on A:

```
Our Application
      |
      v
      A
      |
      v
      B
      |
      v
      C
```

Maven can automatically resolve the required transitive dependencies.

This is called:

> **Transitive Dependency Management**

---

# 14\. Java Reflection API

One of the important technologies behind frameworks such as Spring is Java Reflection.

Reflection allows Java code to inspect classes at runtime.

For example:

```
Class<Student> clazz = Student.class;
```

The variable:

```
clazz
```

contains metadata about the `Student` class.

---

# 15\. What Is `Class` in Java?

Consider:

```
class Student {

    private String name;
    private int age;

    public Student() {
    }

    public void study() {
    }
}
```

Java maintains runtime metadata describing this class.

We can access it using:

```
Class<Student> clazz =
        Student.class;
```

We can inspect:

```
clazz.getName();
```

Fields:

```
clazz.getDeclaredFields();
```

Constructors:

```
clazz.getDeclaredConstructors();
```

Methods:

```
clazz.getDeclaredMethods();
```

Annotations:

```
clazz.getDeclaredAnnotations();
```

---

# 16\. Reflection Example

```
class Student {

    private String name;
    private int age;

    public Student() {
    }

    public void study() {
        System.out.println("Studying");
    }
}
```

Reflection:

```
public class Main {

    public static void main(String[] args) {

        Class<Student> clazz =
                Student.class;

        System.out.println(
                clazz.getName()
        );

        System.out.println(
                clazz.getDeclaredFields()
        );

        System.out.println(
                clazz.getDeclaredConstructors()
        );

        System.out.println(
                clazz.getDeclaredMethods()
        );
    }
}
```

Conceptually:

```
Student.class
     |
     v
+----------------------------+
| Runtime Metadata            |
+----------------------------+
| Class name                  |
| Fields                      |
| Constructors                |
| Methods                     |
| Annotations                 |
| Modifiers                   |
+----------------------------+
```

---

# 17\. Why Does Spring Need Reflection?

Suppose Spring discovers:

```
@Component
public class PaymentService {

}
```

Spring needs to understand:

- What class is this?
- What constructors does it have?
- What annotations exist?
- What dependencies does it need?
- What bean name should it have?
- What scope should it use?

Reflection helps Spring inspect runtime metadata.

However, an important interview correction:

> **Spring does not simply "use reflection to do everything."**

Spring uses a combination of:

- reflection
- metadata processing
- bean definitions
- post-processors
- dependency resolution
- factory mechanisms
- proxies
- configuration processing

Reflection is an important mechanism, but Spring is much more sophisticated than simply calling reflection APIs.

---

# 18\. Spring Bean

A **Spring Bean** is an object whose lifecycle is managed by the Spring IoC container.

For example:

```
@Component
public class PaymentService {

}
```

Spring discovers this class and creates/manages an instance.

Conceptually:

```
PaymentService.class
       |
       | discovered by Spring
       v
Spring Container
       |
       | creates
       v
PaymentService object
       |
       | managed by
       v
Spring Bean
```

---

# 19\. `@Component`

`@Component` tells Spring:

> "This class is a candidate for component scanning and can be registered as a bean."

Example:

```
@Component
public class PaymentService {

    public void pay() {
        System.out.println("Payment Done");
    }
}
```

Another class:

```
@Component
public class OrderService {

    private final PaymentService paymentService;

    public OrderService(
            PaymentService paymentService) {

        this.paymentService =
                paymentService;
    }

    public void placeOrder() {

        paymentService.pay();

        System.out.println(
                "Order Placed"
        );
    }
}
```

---

# 20\. Configuration Class

We need to tell Spring where to scan.

```
@Configuration
@ComponentScan("in.coderarmy")
public class AppConfig {

}
```

Two important annotations:

```
@Configuration
       |
       v
This class contains configuration information

@ComponentScan
       |
       v
Searches packages for component classes
```

---

# 21\. `@Configuration`

Example:

```
@Configuration
public class AppConfig {

}
```

It tells Spring:

> This class provides configuration metadata for the application context.

It is commonly used with:

```
@Bean
```

and can also be used with:

```
@ComponentScan
```

---

# 22\. `@ComponentScan`

Example:

```
@ComponentScan("in.coderarmy")
```

Spring scans:

```
in.coderarmy
      |
      +---- OrderService
      |
      +---- PaymentService
      |
      +---- UserService
      |
      +---- repository
      |       |
      |       +---- UserRepository
      |
      +---- payment
              |
              +---- CardPayment
```

It can discover classes annotated with stereotype annotations such as:

```
@Component
@Service
@Repository
@Controller
```

These are specialized Spring stereotypes built around component registration.

---

# 23\. Starting the Spring Container

We can create an application context:

```
ApplicationContext context =
        new AnnotationConfigApplicationContext(
                AppConfig.class
        );
```

Conceptually:

```
                AppConfig.class
                      |
                      v
       AnnotationConfigApplicationContext
                      |
                      v
              Spring Container
                      |
       +--------------+--------------+
       |                             |
       v                             v
PaymentService                 OrderService
```

---

# 24\. Getting a Bean

Instead of:

```
OrderService order =
        new OrderService(payment);
```

we use:

```
OrderService order =
        context.getBean(OrderService.class);
```

Spring gives us the managed object.

Complete example:

```
package in.coderarmy;

import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;

public class Main {

    public static void main(String[] args) {

        ApplicationContext context =
                new AnnotationConfigApplicationContext(
                        AppConfig.class
                );

        OrderService order =
                context.getBean(OrderService.class);

        order.placeOrder();
    }
}
```

---

# 25\. Complete Initial Spring Example

## PaymentService

```
package in.coderarmy;

import org.springframework.stereotype.Component;

@Component
public class PaymentService {

    public void pay() {
        System.out.println("Payment Done");
    }
}
```

## OrderService

```
package in.coderarmy;

import org.springframework.stereotype.Component;

@Component
public class OrderService {

    private final PaymentService paymentService;

    public OrderService(
            PaymentService paymentService) {

        this.paymentService =
                paymentService;
    }

    public void placeOrder() {

        paymentService.pay();

        System.out.println(
                "Order Placed"
        );
    }
}
```

## AppConfig

```
package in.coderarmy;

import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

@Configuration
@ComponentScan("in.coderarmy")
public class AppConfig {

}
```

## Main

```
package in.coderarmy;

import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;

public class Main {

    public static void main(String[] args) {

        ApplicationContext context =
                new AnnotationConfigApplicationContext(
                        AppConfig.class
                );

        OrderService order =
                context.getBean(OrderService.class);

        order.placeOrder();
    }
}
```

Output:

```
Payment Done
Order Placed
```

---

# 26\. Dependency Injection Types

Spring supports three commonly discussed forms of dependency injection:

```
                 Dependency Injection
                         |
          +--------------+--------------+
          |              |              |
          v              v              v
    Constructor       Setter          Field
       DI               DI              DI
```

The three are:

1. Constructor Injection
2. Setter Injection
3. Field Injection

---

# 27\. Constructor Injection

Example:

```
@Component
public class OrderService {

    private final PaymentService paymentService;

    public OrderService(
            PaymentService paymentService) {

        this.paymentService =
                paymentService;
    }
}
```

Spring sees:

```
OrderService constructor
        |
        v
PaymentService required
        |
        v
Spring searches container
        |
        v
PaymentService bean found
        |
        v
OrderService created
```

---

# 28\. Why Constructor Injection Is Recommended

## 28.1 Immutability

You can use:

```
private final PaymentService paymentService;
```

Once initialized:

```
OrderService
     |
     +---- paymentService
              |
              v
        PaymentService
```

The reference cannot be reassigned.

---

# 29\. Constructor Injection Prevents Invalid Objects

Suppose:

```
public class OrderService {

    private final PaymentService paymentService;

    public OrderService(
            PaymentService paymentService) {

        this.paymentService =
                paymentService;
    }
}
```

You cannot accidentally create:

```
new OrderService(null);
```

Well, technically Java allows explicitly passing `null`, but the class's API clearly requires the dependency and can reject invalid values if needed.

The important architectural point is:

> The object cannot be normally constructed without supplying its required dependency.

---

# 30\. Constructor Injection and Unit Testing

Constructor injection makes testing easy.

Suppose:

```
PaymentService payment =
        new FakePaymentService();

OrderService order =
        new OrderService(payment);
```

No Spring container is required.

This is a major advantage.

---

# 31\. Is `@Autowired` Required on a Constructor?

Suppose there is exactly one constructor:

```
@Component
public class OrderService {

    private final PaymentService paymentService;

    public OrderService(
            PaymentService paymentService) {

        this.paymentService =
                paymentService;
    }
}
```

Modern Spring can automatically use that constructor.

Therefore:

```
@Autowired
public OrderService(...)
```

is generally unnecessary when the class has a single constructor.

---

# 32\. When Is `@Autowired` Useful on Constructors?

If there are multiple constructors:

```
@Component
public class OrderService {

    public OrderService() {
    }

    @Autowired
    public OrderService(
            PaymentService paymentService) {

        // ...
    }
}
```

`@Autowired` can tell Spring which constructor should be used.

---

# 33\. Setter Injection

Example:

```
@Component
public class OrderService {

    private PaymentService paymentService;

    @Autowired
    public void setPaymentService(
            PaymentService paymentService) {

        this.paymentService =
                paymentService;
    }
}
```

Conceptually:

```
Spring creates OrderService
           |
           v
OrderService object
           |
           v
Spring resolves PaymentService
           |
           v
setPaymentService(...)
           |
           v
Dependency injected
```

---

# 34\. When Is Setter Injection Useful?

Setter injection can be appropriate for:

- optional dependencies
- dependencies that can be changed
- legacy code
- configuration that may be supplied after construction

But required dependencies are usually better represented using constructors.

---

# 35\. Field Injection

Example:

```
@Component
public class OrderService {

    @Autowired
    private PaymentService paymentService;
}
```

Spring uses reflection and dependency injection machinery to populate the field.

---

# 36\. Why Is Field Injection Generally Discouraged?

Consider:

```
OrderService order =
        new OrderService();
```

The dependency is not supplied.

So:

```
order.placeOrder();
```

may fail because:

```
paymentService == null
```

With constructor injection:

```
OrderService order =
        new OrderService(paymentService);
```

the dependency is explicit.

---

# 37\. Constructor vs Setter vs Field

| Feature | Constructor | Setter | Field |
| --- | --- | --- | --- |
| Required dependency | Excellent | Not ideal | Not ideal |
| Optional dependency | Possible | Good | Possible |
| Immutability | Excellent | No | No |
| Easy unit testing | Excellent | Good | Poor |
| Dependency visible in API | Yes | Yes | No |
| `final` supported | Yes | No | No |
| Generally recommended | Yes | Sometimes | Usually discouraged |

---

# 38\. Important Interview Question

### Q: Which type of dependency injection is recommended in Spring?

**Answer:**

> Constructor injection is generally recommended for required dependencies because it makes dependencies explicit, supports immutability through `final`, ensures required dependencies are available during construction, and makes unit testing easier without requiring the Spring container.

---

# 39\. Spring's Bean Creation Flow

A simplified view:

```
Application starts
       |
       v
Create ApplicationContext
       |
       v
Read configuration
       |
       v
Process @ComponentScan
       |
       v
Discover candidate classes
       |
       v
Create BeanDefinitions
       |
       v
Resolve dependencies
       |
       v
Instantiate beans
       |
       v
Inject dependencies
       |
       v
Run lifecycle callbacks/post-processors
       |
       v
Beans ready
```

---

# 40\. BeanDefinition

Before Spring creates an actual object, it maintains metadata describing how that bean should be created and managed.

That metadata is represented by a:

```
BeanDefinition
```

Conceptually:

```
BeanDefinition
+---------------------------+
| Bean class                |
| Bean name                 |
| Scope                     |
| Lazy initialization       |
| Dependency metadata       |
| Autowire information      |
| Init method               |
| Destroy method            |
+---------------------------+
```

Think of it as:

> **Instructions/metadata describing a bean.**

---

# 41\. Important Distinction: BeanDefinition vs Bean

These are not the same thing.

```
BeanDefinition
       |
       | describes
       v
Bean instance
```

For example:

```
BeanDefinition:
    class = PaymentService
    scope = singleton
    name = paymentService
```

Later:

```
PaymentService object
```

is instantiated.

---

# 42\. Does Spring Always Instantiate Dependencies in a Simple Linear Order?

Not necessarily.

A simplified teaching diagram might say:

```
PaymentService
      |
      v
OrderService
```

But actual Spring dependency resolution can involve:

- dependency graphs
- multiple levels
- circular dependencies
- lazy beans
- factory methods
- scopes
- proxies
- post-processors

So a more accurate model is:

```
                 ApplicationContext
                        |
                 Dependency Graph
                        |
        +---------------+---------------+
        |               |               |
        v               v               v
   Repository       PaymentService   UserService
        |               |
        v               v
   Database        PaymentGateway
                        |
                        v
                  OrderService
```

Spring resolves the dependency graph according to its container rules.

---

# 43\. Interface-Based Design

Suppose we have:

```
public interface PaymentService {

    void pay();
}
```

Now we create multiple implementations.

---

# 44\. Card Payment

```
@Component
public class CardPayment
        implements PaymentService {

    @Override
    public void pay() {

        System.out.println(
                "Paying via Card"
        );
    }
}
```

---

# 45\. UPI Payment

```
@Component
public class UpiPayment
        implements PaymentService {

    @Override
    public void pay() {

        System.out.println(
                "Paying via UPI"
        );
    }
}
```

Now:

```
              PaymentService
                  interface
                     |
          +----------+----------+
          |                     |
          v                     v
   CardPayment             UpiPayment
```

---

# 46\. OrderService Depends on Interface

```
@Component
public class OrderService {

    private final PaymentService paymentService;

    public OrderService(
            PaymentService paymentService) {

        this.paymentService =
                paymentService;
    }

    public void placeOrder() {

        paymentService.pay();

        System.out.println(
                "Order Placed"
        );
    }
}
```

Architecturally, this is better:

```
OrderService
     |
     | depends on abstraction
     v
PaymentService
     ^
     |
 +---+---+
 |       |
Card    UPI
```

`OrderService` does not care about the concrete implementation.

---

# 47\. The Ambiguity Problem

Now Spring sees:

```
PaymentService
     |
     +---- CardPayment
     |
     +---- UpiPayment
```

Both are beans.

Spring receives:

```
PaymentService paymentService
```

Question:

> Which implementation should Spring inject?

There are two candidates.

Spring cannot safely guess.

This can result in an exception such as:

```
NoUniqueBeanDefinitionException
```

Conceptually:

```
Required:
PaymentService

Found:
1. cardPayment
2. upiPayment

Result:
AMBIGUOUS DEPENDENCY
```

---

# 48\. Solution 1: `@Primary`

We can designate one implementation as the default.

```
@Component
@Primary
public class UpiPayment
        implements PaymentService {

    @Override
    public void pay() {
        System.out.println(
                "Paying via UPI"
        );
    }
}
```

Now:

```
PaymentService
     |
     +---- CardPayment
     |
     +---- UpiPayment ⭐ PRIMARY
```

When Spring sees:

```
PaymentService paymentService
```

it can choose:

```
UpiPayment
```

---

# 49\. What Does `@Primary` Mean?

A good interview definition:

> `@Primary` tells Spring which bean should be preferred when multiple beans are candidates for the same dependency and no more specific selection rule identifies another bean.

It does **not** mean:

> "Only this bean exists."

The other beans still exist.

---

# 50\. Solution 2: `@Qualifier`

Sometimes we don't want a default.

We want a specific implementation.

```
@Component
public class OrderService {

    private final PaymentService paymentService;

    public OrderService(
            @Qualifier("cardPayment")
            PaymentService paymentService) {

        this.paymentService =
                paymentService;
    }
}
```

Now Spring knows:

```
PaymentService
      |
      +---- cardPayment  <-- selected
      |
      +---- upiPayment
```

---

# 51\. `@Primary` vs `@Qualifier`

| Feature | `@Primary` | `@Qualifier` |
| --- | --- | --- |
| Purpose | Default preference | Explicit selection |
| Number of beans | Multiple | Multiple |
| Injection | Implicit preference | Explicit target |
| Best use | Common/default implementation | Specific implementation |

Example:

```
3 Payment implementations

Card
UPI       <-- @Primary
Wallet
```

Normal injection:

```
PaymentService paymentService
```

selects:

```
UPI
```

Explicit:

```
@Qualifier("walletPayment")
PaymentService paymentService
```

selects:

```
Wallet
```

---

# 52\. Default Bean Names

For:

```
@Component
public class CardPayment {
}
```

the default bean name is typically:

```
cardPayment
```

For:

```
@Component
public class UpiPayment {
}
```

typically:

```
upiPayment
```

The conventional default naming strategy uses the uncapitalized short class name.

---

# 53\. Custom Bean Names

You can explicitly specify:

```
@Component("upi")
public class UpiPayment
        implements PaymentService {
}
```

Then:

```
@Qualifier("upi")
```

can target it.

Example:

```
public OrderService(
        @Qualifier("upi")
        PaymentService paymentService) {

    this.paymentService =
            paymentService;
}
```

---

# 54\. Important Interview Trap

### Question:

If I have:

```
@Component("payment")
public class CardPayment {
}
```

and:

```
@Qualifier("cardPayment")
```

will Spring find it?

**Answer:**

No.

The bean name is:

```
payment
```

not:

```
cardPayment
```

Therefore:

```
@Qualifier("payment")
```

would target it by name.

---

# 55\. What Is the Limitation of `@Component`?

`@Component` is convenient:

```
@Component
public class PaymentService {
}
```

But there are situations where it is not appropriate.

Two major examples:

1. The class requires configuration/custom construction logic.
2. The class belongs to a third-party library that you cannot modify.

---

# 56\. Complex Constructors

Consider:

```
public class User {

    private final String name;
    private final int age;

    public User(
            String name,
            int age) {

        this.name = name;
        this.age = age;
    }
}
```

Suppose we write:

```
@Component
public class User {

    public User(
            String name,
            int age) {
    }
}
```

Spring doesn't automatically know:

```
What String should I use?
What int should I use?
```

Spring isn't supposed to invent values.

We need configuration.

---

# 57\. Third-Party Classes

Suppose a library gives us:

```
public class CartService {

    public CartService() {
    }

    public void addItem() {
    }
}
```

The source code is not ours.

We cannot do:

```
@Component
public class CartService {
}
```

because we cannot modify the third-party class.

So how can Spring manage it?

Enter:

```
@Bean
```

---

# 58\. `@Bean`

`@Bean` is used on a method inside configuration.

Example:

```
@Configuration
public class AppConfig {

    @Bean
    public CartService cartService() {

        return new CartService();
    }
}
```

Conceptually:

```
AppConfig
    |
    | executes @Bean method
    v
new CartService()
    |
    v
Spring Container
    |
    v
Managed CartService Bean
```

---

# 59\. `@Component` vs `@Bean`

This distinction is extremely important for interviews.

## `@Component`

```
@Component
public class PaymentService {
}
```

You annotate the class.

Spring discovers the class through component scanning.

## `@Bean`

```
@Configuration
public class AppConfig {

    @Bean
    public PaymentService paymentService() {
        return new PaymentService();
    }
}
```

You annotate a method.

The method explicitly defines how the object should be created.

---

# 60\. Mental Model

Remember this:

```
@Component

"I am a component.
Spring, discover me."
```

Whereas:

```
@Bean

"Spring, use this method's
returned object as a bean."
```

---

# 61\. When Should You Prefer `@Component`?

Use `@Component` and related stereotypes when:

- you own the class
- the class is naturally a Spring-managed application component
- construction is straightforward
- component scanning makes sense

Typical examples:

```
@Service
@Repository
@Controller
@Component
```

---

# 62\. When Should You Prefer `@Bean`?

Use `@Bean` when:

- you need custom construction logic
- the object requires explicit configuration
- you need to construct a third-party object
- you want precise control over bean creation
- the class cannot be modified

Example:

```
@Bean
public ObjectMapper objectMapper() {

    ObjectMapper mapper =
            new ObjectMapper();

    // custom configuration

    return mapper;
}
```

---

# 63\. `@Bean` with Constructor Dependencies

Suppose:

```
public class OrderService {

    private final PaymentService paymentService;

    public OrderService(
            PaymentService paymentService) {

        this.paymentService =
                paymentService;
    }
}
```

We can define:

```
@Configuration
public class AppConfig {

    @Bean
    public PaymentService paymentService() {

        return new PaymentService();
    }

    @Bean
    public OrderService orderService(
            PaymentService paymentService) {

        return new OrderService(
                paymentService
        );
    }
}
```

Spring resolves the method parameter:

```
orderService(PaymentService)
             |
             v
Spring finds PaymentService bean
             |
             v
passes it into method
             |
             v
new OrderService(paymentService)
```

---

# 64\. Why Does Spring Inject Parameters into `@Bean` Methods?

Because a `@Bean` method itself participates in Spring's configuration processing.

For example:

```
@Bean
public OrderService orderService(
        PaymentService paymentService) {

    return new OrderService(
            paymentService
    );
}
```

Spring sees:

```
Bean required:
PaymentService
```

It resolves that dependency from the container.

---

# 65\. Complete `@Bean` Example

Suppose:

```
public class User {

    private final String name;
    private final int age;

    public User(String name, int age) {

        this.name = name;
        this.age = age;
    }

    public void printUser() {

        System.out.println(
                name + " " + age
        );
    }
}
```

Configuration:

```
@Configuration
public class AppConfig {

    @Bean
    public User user() {

        return new User(
                "Aditya",
                28
        );
    }
}
```

Spring now manages the returned object.

---

# 66\. Third-Party Bean Example

Suppose an external dependency contains:

```
public class CartService {

    public void addItem() {

        System.out.println(
                "Item added"
        );
    }
}
```

We cannot annotate it.

Instead:

```
@Configuration
public class AppConfig {

    @Bean
    public CartService cartService() {

        return new CartService();
    }
}
```

Now:

```
@Component
public class OrderService {

    private final CartService cartService;

    public OrderService(
            CartService cartService) {

        this.cartService =
                cartService;
    }
}
```

Spring can inject the third-party object.

---

# 67\. `@Component` and `@Bean` Together

A common interview question:

> What happens if the same logical object is registered through both component scanning and a `@Bean` method?

Be careful with blanket statements such as:

> "`@Bean` always overrides `@Component`."

That is too simplistic.

If both mechanisms register beans, Spring can end up with **multiple bean definitions/beans**, depending on their names and configuration.

For example:

```
@Component
public class PaymentService {
}
```

and:

```
@Bean
public PaymentService paymentService() {

    return new PaymentService();
}
```

can result in separate registrations if their bean names differ.

Therefore the safer interview answer is:

> `@Component` and `@Bean` are two different bean-registration mechanisms. If both register the same type, bean naming and registration rules determine whether they represent separate beans or conflict. Don't assume that `@Bean` universally "overrides" `@Component`.

---

# 68\. Bean Naming Matters

Example:

```
@Component
public class PaymentService {
}
```

usually produces:

```
paymentService
```

A configuration method:

```
@Bean
public PaymentService paymentService() {
    return new PaymentService();
}
```

also naturally uses:

```
paymentService
```

Matching names can therefore become significant.

---

# 69\. ApplicationContext

The main interface commonly used by modern Spring applications is:

```
ApplicationContext
```

Example:

```
ApplicationContext context =
        new AnnotationConfigApplicationContext(
                AppConfig.class
        );
```

It provides access to Spring's container functionality.

---

# 70\. BeanFactory

`BeanFactory` is the foundational Spring IoC container interface.

Conceptually:

```
BeanFactory
     ^
     |
     | extended by
     |
ApplicationContext
```

A simplified mental model:

```
BeanFactory
    |
    +-- Basic bean management
    |
    +-- Object retrieval
    |
    +-- Dependency management
    |
    v
ApplicationContext
    |
    +-- BeanFactory capabilities
    +-- Events
    +-- Resource loading
    +-- Message resolution / i18n
    +-- Integration with other Spring infrastructure
    +-- Application-level features
```

---

# 71\. BeanFactory vs ApplicationContext

| Feature | BeanFactory | ApplicationContext |
| --- | --- | --- |
| Basic IoC | Yes | Yes |
| Bean creation | Yes | Yes |
| Dependency injection | Yes | Yes |
| Events | Basic/no full application-context facility | Yes |
| Internationalization | Limited | Yes |
| Resource loading | Basic | Better support |
| Application integration | Basic | Extensive |
| Typical modern application choice | Rare | Recommended |

---

# 72\. Which One Should You Use?

For most modern Spring applications:

```
ApplicationContext
```

is the normal choice.

Example:

```
ApplicationContext context =
        new AnnotationConfigApplicationContext(
                AppConfig.class
        );
```

---

# 73\. Spring Container Startup — Detailed View

A simplified startup sequence looks like:

```
                    Application starts
                           |
                           v
              Create ApplicationContext
                           |
                           v
                 Read configuration
                           |
                           v
                 Process configuration
                           |
                           v
                  Component scanning
                           |
                           v
                  Find bean candidates
                           |
                           v
                  Register BeanDefinitions
                           |
                           v
                Resolve dependencies
                           |
                           v
                  Create bean instances
                           |
                           v
                 Dependency injection
                           |
                           v
             Bean post-processing/lifecycle
                           |
                           v
                     Ready to use
```

---

# 74\. Step 1 — Start Container

Code:

```
ApplicationContext context =
        new AnnotationConfigApplicationContext(
                AppConfig.class
        );
```

This begins initialization.

---

# 75\. Step 2 — Read Configuration

Spring processes:

```
@Configuration
```

and configuration-related annotations such as:

```
@ComponentScan
@Bean
@Import
```

---

# 76\. Step 3 — Component Scanning

If:

```
@ComponentScan("in.coderarmy")
```

is configured, Spring searches the specified package and relevant subpackages for candidate components.

Example:

```
in.coderarmy
     |
     +-- OrderService       @Component
     |
     +-- PaymentService     @Component
     |
     +-- UserService        @Service
     |
     +-- UserRepository     @Repository
```

---

# 77\. Step 4 — Candidate Discovery

Spring identifies classes that qualify as components.

For example:

```
@Component
public class PaymentService {
}
```

is a candidate.

---

# 78\. Step 5 — BeanDefinition Registration

Spring registers metadata.

Conceptually:

```
Bean name: paymentService
Class: PaymentService
Scope: singleton
Lazy: false
Dependencies: none
```

This becomes part of the container's bean-definition registry.

---

# 79\. Step 6 — Bean Creation and Wiring

Spring creates required objects.

Example:

```
PaymentService
      |
      v
PaymentService instance

OrderService requires PaymentService
      |
      v
Spring resolves dependency
      |
      v
OrderService instance
```

---

# 80\. Step 7 — Ready for Application Use

Now:

```
OrderService order =
        context.getBean(OrderService.class);
```

returns the managed bean.

---

# 81\. Important: Spring Is Not Just a Map

Beginners often think:

```
ApplicationContext = HashMap<String, Object>
```

Conceptually, thinking of a container as a registry can help, but Spring is far more sophisticated.

It manages:

- definitions
- scopes
- lifecycle
- dependency resolution
- post-processors
- proxies
- configuration
- events
- resources
- environment
- bean factories

So avoid saying in interviews:

> "Spring is just a map containing objects."

That's an oversimplification.

---

# 82\. Spring Bean Lifecycle

A simplified lifecycle:

```
Bean Definition
      |
      v
Instantiate
      |
      v
Populate dependencies
      |
      v
Aware callbacks
      |
      v
BeanPostProcessor - before initialization
      |
      v
Initialization callbacks
      |
      v
BeanPostProcessor - after initialization
      |
      v
Ready
      |
      v
Destroy
```

This becomes particularly important in advanced Spring interviews.

---

# 83\. Why Bean Lifecycle Matters

Suppose you want to:

- open a connection
- initialize a resource
- validate configuration
- perform startup logic
- clean up resources

Spring provides lifecycle mechanisms for these tasks.

Common annotations include:

```
@PostConstruct
@PreDestroy
```

Example:

```
@Component
public class DatabaseService {

    @PostConstruct
    public void init() {

        System.out.println(
                "DatabaseService initialized"
        );
    }

    @PreDestroy
    public void destroy() {

        System.out.println(
                "DatabaseService destroyed"
        );
    }
}
```

---

# 84\. Bean Scope

Another important Spring concept is bean scope.

The most common scope is:

```
singleton
```

Conceptually:

```
getBean()
    |
    +------------------+
    |                  |
    v                  v
same bean         same bean
instance          instance
```

Other scopes include:

```
singleton
prototype
request
session
application
websocket
```

Web-specific scopes require a suitable web-aware context.

---

# 85\. Singleton Does Not Mean Java Singleton

Important interview distinction.

Spring's:

```
singleton scope
```

means:

> One bean instance per Spring container.

It does not necessarily mean:

> One instance for the entire JVM under every possible context.

For example, two separate Spring application contexts can have separate singleton instances.

---

# 86\. Dependency Injection with Interfaces

A production-style design often looks like:

```
              OrderService
                   |
                   v
          PaymentProcessor
              interface
                   ^
          +--------+--------+
          |                 |
          v                 v
     CardProcessor      UpiProcessor
```

The service depends on:

```
PaymentProcessor
```

not:

```
CardProcessor
```

This gives:

- loose coupling
- easier testing
- easier replacement
- extensibility
- cleaner architecture

---

# 87\. Real-World Example

Imagine an online shopping system.

```
OrderService
    |
    +---- PaymentProcessor
    |
    +---- InventoryService
    |
    +---- NotificationService
```

Payment:

```
PaymentProcessor
     |
     +---- CardPayment
     +---- UpiPayment
     +---- NetBankingPayment
```

Notification:

```
NotificationService
     |
     +---- EmailNotification
     +---- SmsNotification
     +---- PushNotification
```

Spring can wire these implementations based on configuration.

---

# 88\. Interview Problem: Multiple Implementations

### Problem

You have:

```
public interface PaymentService {
    void pay();
}
```

and:

```
@Component
public class CardPayment
        implements PaymentService {
}
```

```
@Component
public class UpiPayment
        implements PaymentService {
}
```

Then:

```
@Component
public class OrderService {

    private final PaymentService paymentService;

    public OrderService(
            PaymentService paymentService) {
        this.paymentService =
                paymentService;
    }
}
```

### What happens?

Spring finds two candidates.

Result:

```
NoUniqueBeanDefinitionException
```

### Solutions

Use:

```
@Primary
```

or:

```
@Qualifier
```

---

# 89\. Interview Problem: `@Primary`

### Question

What happens here?

```
@Component
@Primary
class CardPayment
        implements PaymentService {
}
```

```
@Component
class UpiPayment
        implements PaymentService {
}
```

and:

```
public OrderService(
        PaymentService paymentService) {
}
```

### Answer

Spring chooses:

```
CardPayment
```

because it is marked `@Primary`.

---

# 90\. Interview Problem: `@Qualifier`

### Question

What happens here?

```
@Component
class CardPayment
        implements PaymentService {
}
```

```
@Component
class UpiPayment
        implements PaymentService {
}
```

and:

```
public OrderService(
        @Qualifier("upiPayment")
        PaymentService paymentService) {
}
```

### Answer

Spring selects:

```
UpiPayment
```

provided the bean name is indeed `upiPayment`.

---

# 91\. Interview Problem: Primary + Qualifier

Suppose:

```
@Component
@Primary
class UpiPayment
        implements PaymentService {
}
```

and:

```
@Component
class CardPayment
        implements PaymentService {
}
```

Then:

```
public OrderService(
        @Qualifier("cardPayment")
        PaymentService paymentService) {
}
```

Which one wins?

### Answer

The qualifier explicitly selects:

```
CardPayment
```

The `@Primary` preference does not override an explicit qualifier selection.

Think:

```
@Qualifier
    |
    v
Explicit selection

@Primary
    |
    v
Default selection
```

---

# 92\. Interview Problem: No Bean

Suppose:

```
@Component
public class OrderService {

    private final PaymentService paymentService;

    public OrderService(
            PaymentService paymentService) {
    }
}
```

But there is no bean implementing:

```
PaymentService
```

What happens?

Spring cannot resolve the dependency.

The application context fails during bean creation.

---

# 93\. Interview Problem: Component Scan

Suppose:

```
com.example.config
com.example.service
```

But:

```
@ComponentScan("com.example.config")
```

and `OrderService` is located in:

```
com.example.service
```

What happens?

Spring won't discover `OrderService` through that component scan.

Result:

```
OrderService is not registered
```

A later:

```
context.getBean(OrderService.class)
```

can fail because the bean isn't present.

---

# 94\. Interview Problem: Package Structure

A common setup:

```
in.coderarmy
   |
   +-- AppConfig
   |
   +-- Main
   |
   +-- service
   |     |
   |     +-- OrderService
   |
   +-- payment
         |
         +-- PaymentService
```

Using:

```
@ComponentScan("in.coderarmy")
```

will scan the package and its subpackages.

This is why package placement matters.

---

# 95\. Interview Problem: Missing `@Component`

Suppose:

```
public class PaymentService {
}
```

There is no:

```
@Component
```

and no:

```
@Bean
```

Then component scanning won't automatically register it as a bean.

If `OrderService` requires it:

```
public OrderService(
        PaymentService paymentService) {
}
```

Spring cannot resolve the dependency unless some other configuration registers it.

---

# 96\. Interview Problem: Constructor Injection

### Question

Why is this preferred?

```
@Component
public class OrderService {

    private final PaymentService paymentService;

    public OrderService(
            PaymentService paymentService) {

        this.paymentService =
                paymentService;
    }
}
```

### Answer

Because:

- dependency is explicit
- required dependency is supplied during construction
- supports `final`
- improves immutability
- improves unit testing
- makes the class's requirements visible
- avoids hidden framework-based field injection

---

# 97\. Interview Problem: Field Injection

### Question

Why is this often discouraged?

```
@Autowired
private PaymentService paymentService;
```

### Answer

Because:

- dependency is hidden
- field cannot normally be `final`
- unit testing becomes less convenient
- object can exist in an incompletely initialized state outside Spring
- stronger coupling to framework-based injection is introduced

---

# 98\. Interview Problem: `@Component` vs `@Bean`

### Question

When should you use `@Bean` instead of `@Component`?

### Answer

Use `@Bean` when:

- you don't own the class
- it is a third-party class
- construction requires custom logic
- you need explicit configuration
- you want to configure the returned object before registering it

Example:

```
@Bean
public ObjectMapper objectMapper() {

    ObjectMapper mapper =
            new ObjectMapper();

    // custom configuration

    return mapper;
}
```

---

# 99\. Interview Problem: Third-Party Library

### Question

You have:

```
ExternalClient
```

from a third-party JAR.

You cannot add:

```
@Component
```

How can Spring manage it?

### Answer

Create a configuration:

```
@Configuration
public class AppConfig {

    @Bean
    public ExternalClient externalClient() {

        return new ExternalClient();
    }
}
```

Now Spring manages the returned object.

---

# 100\. Interview Problem: Complex Constructor

Suppose:

```
public class User {

    private final String username;
    private final int age;

    public User(
            String username,
            int age) {

        this.username = username;
        this.age = age;
    }
}
```

How can Spring manage it?

One option:

```
@Configuration
public class AppConfig {

    @Bean
    public User user() {

        return new User(
                "aditya",
                28
        );
    }
}
```

This is much clearer than expecting Spring to somehow guess the primitive values.

---

# 101\. Interview Problem: `ApplicationContext`

### Question

What is `ApplicationContext`?

### Answer

> `ApplicationContext` is a central Spring IoC container interface responsible for managing beans and providing broader application-level container functionality such as event publication, resource loading, message resolution, and integration with Spring infrastructure.

---

# 102\. Interview Problem: BeanFactory

### Question

What is `BeanFactory`?

### Answer

> `BeanFactory` is the foundational Spring IoC container interface providing core bean creation and dependency-management capabilities. `ApplicationContext` builds on it and provides additional application-level features.

---

# 103\. Interview Problem: Why Use ApplicationContext?

### Answer

Because it provides more functionality than basic bean management, including:

- application events
- resource loading
- message resolution
- integration with Spring infrastructure
- broader enterprise application capabilities

For modern Spring applications, `ApplicationContext` is generally the normal choice.

---

# 104\. Important Architecture Diagram

The overall architecture can be remembered like this:

```
                    APPLICATION
                         |
                         v
              +---------------------+
              |  ApplicationContext |
              +----------+----------+
                         |
               +---------+---------+
               |                   |
               v                   v
       Bean Definitions       Configuration
               |                   |
               +---------+---------+
                         |
                         v
                 Dependency Graph
                         |
        +----------------+----------------+
        |                |                |
        v                v                v
 OrderService     PaymentService     UserService
        |                |
        |                |
        v                v
 PaymentService      Gateway
        |
        +-------------------+
        |                   |
        v                   v
   CardPayment          UpiPayment
```

---

# 105\. Manual DI vs Spring DI

## Manual DI

```
PaymentService payment =
        new PaymentService();

OrderService order =
        new OrderService(payment);
```

We control:

```
creation
wiring
lifecycle
```

## Spring DI

```
ApplicationContext context =
        new AnnotationConfigApplicationContext(
                AppConfig.class
        );

OrderService order =
        context.getBean(OrderService.class);
```

Spring controls:

```
creation
wiring
lifecycle
configuration
scope
post-processing
```

---

# 106\. Dependency Inversion Principle

Don't confuse:

```
Dependency Injection
```

with:

```
Dependency Inversion Principle
```

They are related but different.

DIP generally encourages high-level modules to depend on abstractions rather than concrete low-level implementations.

Example:

Bad:

```
OrderService
     |
     v
StripePayment
```

Better:

```
OrderService
     |
     v
PaymentService
   interface
     ^
     |
StripePayment
```

Spring DI makes implementing this architecture much easier.

---

# 107\. Constructor Injection + Interface = Strong Design

A very common production pattern:

```
public interface PaymentService {

    void pay();
}
```

Implementation:

```
@Component
public class UpiPayment
        implements PaymentService {

    @Override
    public void pay() {
        System.out.println(
                "UPI payment"
        );
    }
}
```

Consumer:

```
@Component
public class OrderService {

    private final PaymentService paymentService;

    public OrderService(
            PaymentService paymentService) {

        this.paymentService =
                paymentService;
    }
}
```

The dependency graph becomes:

```
OrderService
     |
     v
PaymentService
     ^
     |
UpiPayment
```

---

# 108\. A More Realistic Payment Architecture

Imagine:

```
                    PaymentService
                          |
              +-----------+-----------+
              |           |           |
              v           v           v
             UPI         Card       Wallet
              |           |           |
              v           v           v
           UPI API     Bank API    Wallet API
```

Then:

```
OrderService
     |
     v
PaymentService
```

`OrderService` doesn't need to know how UPI or Card works.

This is loose coupling.

---

# 109\. Dependency Graph Example

Consider:

```
class OrderService {

    OrderService(
        PaymentService payment,
        InventoryService inventory
    ) {}
}
```

and:

```
class PaymentService {

    PaymentService(
        PaymentGateway gateway
    ) {}
}
```

Then:

```
OrderService
   |
   +---- PaymentService
   |          |
   |          +---- PaymentGateway
   |
   +---- InventoryService
```

Spring recursively resolves these dependencies.

---

# 110\. What If There Is a Circular Dependency?

Example:

```
A -> B
B -> A
```

```
class A {

    A(B b) {
    }
}
```

```
class B {

    B(A a) {
    }
}
```

Conceptually:

```
A
|
v
B
|
v
A
|
v
B
...
```

This is a circular dependency.

Constructor-based circular dependencies are especially problematic because neither object can be fully constructed before the other exists.

This is one reason circular dependencies are generally a design smell and should usually be refactored.

---

# 111\. How Can You Fix Circular Dependencies?

Usually, don't try to "hack" around them first.

Ask:

> Why do A and B need each other?

Often the architecture can be improved.

For example:

```
A <----> B
```

might become:

```
A ---> C <--- B
```

where `C` contains shared responsibilities.

---

# 112\. Common Spring Interview Question

### Q: Can Spring inject an interface?

Yes.

Example:

```
public interface PaymentService {
}
```

and:

```
@Component
public class CardPayment
        implements PaymentService {
}
```

Spring can inject:

```
PaymentService paymentService;
```

because Spring resolves the interface dependency to a suitable bean implementation.

If multiple implementations exist, use:

```
@Primary
```

or:

```
@Qualifier
```

---

# 113\. Common Interview Question

### Q: Can `@Autowired` be used on private fields?

Yes.

For field injection, Spring can inject into private fields using its reflection-based infrastructure.

Example:

```
@Autowired
private PaymentService paymentService;
```

However, this doesn't mean field injection is the recommended style.

Constructor injection is generally preferred for required dependencies.

---

# 114\. Common Interview Question

### Q: Can `@Autowired` be omitted?

For a class with exactly one constructor, yes.

Example:

```
@Component
public class OrderService {

    private final PaymentService paymentService;

    public OrderService(
            PaymentService paymentService) {

        this.paymentService =
                paymentService;
    }
}
```

No `@Autowired` is necessary.

---

# 115\. Common Interview Question

### Q: What happens if there are two constructors?

Example:

```
@Component
public class OrderService {

    public OrderService() {
    }

    public OrderService(
            PaymentService paymentService) {
    }
}
```

Spring needs constructor-selection information according to the applicable constructor/autowiring rules.

A common explicit solution is:

```
@Autowired
public OrderService(
        PaymentService paymentService) {
}
```

---

# 116\. Common Interview Question

### Q: Does `@Component` create the object?

Conceptually yes, but the precise answer is:

> `@Component` marks a class as a component candidate. Spring's container discovers the class, registers its bean definition, and creates/manages the corresponding bean instance according to container configuration and lifecycle rules.

This is more accurate than saying:

> "`@Component` itself creates an object."

Annotations don't execute object creation by themselves.

Spring processes them.

---

# 117\. Common Interview Question

### Q: Does Spring use Reflection?

Yes, Spring heavily uses reflection and runtime metadata processing.

But Spring also uses:

- bean definitions
- post-processors
- factories
- proxies
- dependency resolution algorithms
- configuration processing

So:

> Reflection is one important mechanism inside Spring, not the entire Spring framework.

---

# 118\. Common Interview Question

### Q: What is the difference between an object and a bean?

Every bean is an object, but not every object is necessarily a Spring bean.

Example:

```
PaymentService p =
        new PaymentService();
```

This is an ordinary Java object.

If Spring creates and manages it:

```
Spring Container
      |
      v
PaymentService instance
```

then it is a Spring bean.

---

# 119\. Common Interview Question

### Q: Can a Spring Bean be created manually using `new`?

Yes, Java allows it.

But if you create:

```
new PaymentService();
```

outside the Spring container, that object is not automatically the same managed bean instance and won't automatically receive Spring-managed injection/lifecycle behavior.

This is an important distinction.

---

# 120\. Common Interview Question

### Q: What is Dependency Lookup?

Instead of receiving dependencies automatically, the application asks the container for an object:

```
PaymentService payment =
        context.getBean(PaymentService.class);
```

This is called:

> **Dependency Lookup**

Dependency Injection is different:

```
Dependency Lookup
Application asks Spring for dependency

Dependency Injection
Spring supplies dependency to application object
```

---

# 121\. Dependency Lookup vs Dependency Injection

```
Dependency Lookup:

OrderService
     |
     | asks
     v
ApplicationContext
     |
     v
PaymentService
```

DI:

```
ApplicationContext
      |
      | injects
      v
OrderService
      |
      v
PaymentService
```

DI generally produces cleaner decoupling because the dependent class does not need to know about the container.

---

# 122\. Why Is `context.getBean()` Not Ideal Everywhere?

Consider:

```
public class OrderService {

    public void placeOrder(
            ApplicationContext context) {

        PaymentService payment =
                context.getBean(
                        PaymentService.class
                );
    }
}
```

Now `OrderService` knows about Spring.

That introduces framework/container awareness into business logic.

Better:

```
public class OrderService {

    private final PaymentService paymentService;

    public OrderService(
            PaymentService paymentService) {

        this.paymentService =
                paymentService;
    }
}
```

The container performs the wiring externally.

---

# 123\. Interview Scenario

### Question

You are designing a payment system with:

```
CardPayment
UpiPayment
WalletPayment
```

The business requirement says:

> UPI should be the default payment mechanism, but some services explicitly require Card.

How would you configure it?

### Answer

Mark UPI as:

```
@Primary
```

```
@Component
@Primary
public class UpiPayment
        implements PaymentService {
}
```

And use:

```
@Qualifier("cardPayment")
```

where Card is explicitly required.

Architecture:

```
                    PaymentService
                         |
           +-------------+-------------+
           |             |             |
           v             v             v
         UPI ⭐         Card         Wallet
        PRIMARY
           |
           +--> default injection

Card-specific service
           |
           +--> @Qualifier("cardPayment")
```

---

# 124\. Interview Scenario: Third-Party Client

### Problem

Your company uses a third-party:

```
PaymentGatewayClient
```

You cannot modify it.

It requires:

```
PaymentGatewayClient(
    String apiKey,
    String endpoint
)
```

How would you register it?

### Solution

Use `@Bean`.

Conceptually:

```
@Configuration
public class PaymentConfig {

    @Bean
    public PaymentGatewayClient paymentGatewayClient() {

        return new PaymentGatewayClient(
                "configured-api-key",
                "configured-endpoint"
        );
    }
}
```

In a real application, sensitive/configurable values should come from proper configuration management rather than hardcoding secrets.

---

# 125\. Interview Scenario: Testing

### Problem

You want to unit test:

```
OrderService
```

without starting Spring.

Constructor injection:

```
PaymentService fakePayment =
        new FakePaymentService();

OrderService order =
        new OrderService(fakePayment);
```

This is easy.

Field injection would require manually setting the private field or using reflection/test utilities.

Therefore constructor injection improves testability.

---

# 126\. Interview Scenario: Why Interfaces?

Suppose:

```
class OrderService {

    private final UpiPayment payment;

    public OrderService(
            UpiPayment payment) {
    }
}
```

This tightly couples `OrderService` to UPI.

Better:

```
class OrderService {

    private final PaymentService payment;

    public OrderService(
            PaymentService payment) {
    }
}
```

Now:

```
OrderService
      |
      v
PaymentService
      ^
      |
 +----+----+
 |         |
UPI       Card
```

This follows a more extensible design.

---

# 127\. Coding Interview Problem #1

## Problem

Create:

```
NotificationService
```

with implementations:

```
EmailNotification
SmsNotification
```

Inject the interface into:

```
OrderService
```

and make Email the default.

### Solution

```
public interface NotificationService {

    void send(String message);
}
```

```
@Component
@Primary
public class EmailNotification
        implements NotificationService {

    @Override
    public void send(String message) {

        System.out.println(
                "Email: " + message
        );
    }
}
```

```
@Component
public class SmsNotification
        implements NotificationService {

    @Override
    public void send(String message) {

        System.out.println(
                "SMS: " + message
        );
    }
}
```

```
@Component
public class OrderService {

    private final NotificationService notificationService;

    public OrderService(
            NotificationService notificationService) {

        this.notificationService =
                notificationService;
    }

    public void placeOrder() {

        notificationService.send(
                "Order placed"
        );
    }
}
```

---

# 128\. Coding Interview Problem #2

## Problem

You have:

```
public class User {

    public User(
            String username,
            int age) {
    }
}
```

Register it as a Spring bean.

### Answer

```
@Configuration
public class AppConfig {

    @Bean
    public User user() {

        return new User(
                "coder",
                25
        );
    }
}
```

---

# 129\. Coding Interview Problem #3

## Problem

A third-party class is:

```
public class ReportClient {

    public ReportClient(
            String url) {
    }
}
```

You cannot modify it.

Register it with Spring.

### Answer

```
@Configuration
public class AppConfig {

    @Bean
    public ReportClient reportClient() {

        return new ReportClient(
                "https://example.com"
        );
    }
}
```

In production, the URL should normally be externalized into configuration.

---

# 130\. Coding Interview Problem #4

## Problem

There are three implementations:

```
CreditCardPayment
UpiPayment
WalletPayment
```

UPI is the default.

A specific service must use Wallet.

### Solution

```
@Component
@Primary
public class UpiPayment
        implements PaymentService {
}
```

Then:

```
@Component
public class WalletOrderService {

    private final PaymentService paymentService;

    public WalletOrderService(
            @Qualifier("walletPayment")
            PaymentService paymentService) {

        this.paymentService =
                paymentService;
    }
}
```

---

# 131\. Coding Interview Problem #5

## Problem

Find the issue:

```
@Component
public class A {

    private B b;

    public A() {
        b = null;
    }
}
```

### Answer

The dependency is not injected.

Better:

```
@Component
public class A {

    private final B b;

    public A(B b) {

        this.b = b;
    }
}
```

---

# 132\. Coding Interview Problem #6

## Problem

What is wrong here?

```
@Component
public class OrderService {

    private final PaymentService paymentService;

    public OrderService() {
    }
}
```

### Answer

The `final` field is never initialized.

More importantly, Spring has no constructor parameter through which the required dependency can be supplied.

Correct:

```
@Component
public class OrderService {

    private final PaymentService paymentService;

    public OrderService(
            PaymentService paymentService) {

        this.paymentService =
                paymentService;
    }
}
```

---

# 133\. Coding Interview Problem #7

## Problem

What happens?

```
@Component
class CardPayment
        implements PaymentService {
}
```

```
@Component
class UpiPayment
        implements PaymentService {
}
```

```
@Component
class OrderService {

    OrderService(
            PaymentService paymentService) {
    }
}
```

### Answer

There are multiple candidates.

Spring cannot determine the unique implementation.

Use:

```
@Primary
```

or:

```
@Qualifier
```

---

# 134\. Coding Interview Problem #8

## Problem

What happens if the dependency isn't inside the component scan package?

Example:

```
com.example.service.OrderService
com.example.payment.PaymentService
```

but:

```
@ComponentScan(
    "com.example.service"
)
```

### Answer

`PaymentService` isn't discovered through that scan unless it is registered through another configuration mechanism.

Therefore injection may fail.

---

# 135\. 25 Important Spring Core Interview Questions

## Basic

1. What is Spring?
2. What is IoC?
3. What is Dependency Injection?
4. Difference between IoC and DI?
5. What is a Spring Bean?
6. What is `ApplicationContext`?
7. What is `BeanFactory`?
8. What is component scanning?
9. What does `@Component` do?
10. What does `@Configuration` do?

## Dependency Injection

11. What are the types of DI?
12. Why is constructor injection preferred?
13. Is `@Autowired` required on a single constructor?
14. What is field injection?
15. Why is field injection discouraged?
16. What is setter injection?
17. Can Spring inject interfaces?

## Multiple Beans

18. What is `@Primary`?
19. What is `@Qualifier`?
20. Difference between `@Primary` and `@Qualifier`?
21. What happens when multiple beans match a dependency?
22. How does Spring determine a default bean name?

## Configuration

23. Difference between `@Component` and `@Bean`?
24. When should `@Bean` be used?
25. How can a third-party class be registered as a Spring bean?

---

# 136\. Rapid-Fire Interview Answers

### What is IoC?

> IoC is a principle where control over object creation and dependency management is transferred from application code to a container/framework.

### What is DI?

> DI is a technique for supplying an object's dependencies from outside rather than having the object create them itself.

### What is a Spring Bean?

> An object whose lifecycle and management are handled by the Spring IoC container.

### What is `@Component`?

> A class-level stereotype annotation that makes a class eligible for component scanning and Spring bean registration.

### What is `@Bean`?

> A method-level annotation used to explicitly register the object returned by a configuration method as a Spring bean.

### What is `@Autowired`?

> It tells Spring to resolve and inject a suitable dependency at an injection point.

### What is `@Primary`?

> It marks a bean as the preferred candidate when multiple beans match a dependency.

### What is `@Qualifier`?

> It provides a more explicit way to select a particular bean among multiple candidates.

### What is `ApplicationContext`?

> A higher-level Spring IoC container interface that extends the core bean-factory capabilities and provides additional application-level features.

### What is `BeanFactory`?

> The foundational Spring IoC container interface providing core bean-management capabilities.

---

# 137\. Important Misconceptions to Avoid

## Misconception 1

> "`@Autowired` always needs to be written."

False.

With one constructor, it can generally be omitted.

---

## Misconception 2

> "`@Component` means Spring immediately creates an object."

Oversimplified.

It marks the class as a component candidate. Spring processes the metadata and creates/manages the bean according to its container lifecycle and configuration.

---

## Misconception 3

> "`@Primary` disables all other beans."

False.

Other beans still exist.

`@Primary` simply provides a preferred candidate during ambiguous autowiring.

---

## Misconception 4

> "`@Qualifier` creates a new bean."

False.

It selects an existing matching bean.

---

## Misconception 5

> "`@Bean` can be placed anywhere."

Not as a general rule.

It is normally used on methods processed as bean definitions, most commonly inside `@Configuration` classes.

---

## Misconception 6

> "`ApplicationContext` is just a HashMap."

False.

A registry is a useful mental model, but Spring's container performs extensive lifecycle, dependency-resolution, post-processing, configuration, and infrastructure work.

---

## Misconception 7

> "Spring uses only reflection."

False.

Reflection is important, but Spring uses many mechanisms.

---

# 138\. Full End-to-End Mental Model

The complete journey is:

```
                 YOUR APPLICATION
                       |
                       v
              AppConfig.class
                       |
                       v
            ApplicationContext
                       |
                       v
               Component Scan
                       |
                       v
             Discover Components
                       |
                       v
              BeanDefinitions
                       |
                       v
             Dependency Graph
                       |
                       v
             Bean Instantiation
                       |
                       v
             Dependency Injection
                       |
                       v
              Bean Post-Processing
                       |
                       v
               Lifecycle callbacks
                       |
                       v
                 READY BEANS
                       |
                       v
               Application Usage
```

---

# 139\. Complete Architecture

```
                         Main
                          |
                          |
                          v
                +-------------------+
                | ApplicationContext|
                +---------+---------+
                          |
                          |
                reads AppConfig
                          |
                          v
                +-------------------+
                |   Component Scan  |
                +---------+---------+
                          |
             +------------+------------+
             |                         |
             v                         v
       PaymentService             OrderService
       @Component                 @Component
             |                         |
             |                         |
             +----------+--------------+
                        |
                        v
                Dependency Resolution
                        |
                        v
                 Fully Wired Beans
```

---

# 140\. The Evolution of Our Code

## Version 1 — Tight Coupling

```
public class OrderService {

    private PaymentService paymentService =
            new PaymentService();
}
```

Problem:

```
OrderService ---> concrete PaymentService
```

---

## Version 2 — Manual DI

```
public class OrderService {

    private final PaymentService paymentService;

    public OrderService(
            PaymentService paymentService) {

        this.paymentService =
                paymentService;
    }
}
```

Creation:

```
PaymentService payment =
        new PaymentService();

OrderService order =
        new OrderService(payment);
```

Better.

---

## Version 3 — Spring DI

```
@Component
public class OrderService {

    private final PaymentService paymentService;

    public OrderService(
            PaymentService paymentService) {

        this.paymentService =
                paymentService;
    }
}
```

Spring handles:

```
Creation
   +
Resolution
   +
Injection
   +
Lifecycle
```

---

## Version 4 — Interface + Multiple Implementations

```
OrderService
     |
     v
PaymentService
     ^
     |
 +---+---+
 |       |
Card    UPI
```

Then:

```
@Primary
```

or:

```
@Qualifier
```

solves ambiguity.

---

# 141\. The Most Important Annotations

```
@Configuration
       |
       v
Configuration class

@ComponentScan
       |
       v
Find component classes

@Component
       |
       v
Register component candidate

@Bean
       |
       v
Explicitly register method return object

@Autowired
       |
       v
Inject dependency

@Primary
       |
       v
Preferred candidate

@Qualifier
       |
       v
Specific candidate
```

---

# 142\. One-Page Interview Cheat Sheet

```
IoC
|
+-- Control moves from application -> container
|
DI
|
+-- Dependencies supplied externally
|
+-- Constructor
+-- Setter
+-- Field
|
@Component
|
+-- Class-level
+-- Component scanning
|
@Bean
|
+-- Method-level
+-- Explicit bean creation
+-- Great for third-party classes
|
@Autowired
|
+-- Dependency injection
+-- Optional on single constructor
|
@Primary
|
+-- Default/preferred bean
|
@Qualifier
|
+-- Explicit bean selection
|
ApplicationContext
|
+-- Modern application-level IoC container
|
BeanFactory
|
+-- Foundational IoC container interface
|
Reflection
|
+-- Runtime class metadata/introspection
|
BeanDefinition
|
+-- Metadata describing how Spring manages a bean
```

---

# 143\. Final Interview-Level Summary

If an interviewer asks:

> "Explain Spring IoC and Dependency Injection."

A strong answer would be:

> Spring's IoC container manages the creation and lifecycle of application objects, called beans. Instead of a class creating its own dependencies using `new`, dependencies can be supplied externally through Dependency Injection. Constructor injection is generally preferred because it makes required dependencies explicit, supports immutability, and improves testability. Spring discovers components through mechanisms such as component scanning, builds bean definitions, resolves dependencies, creates and wires beans, and manages their lifecycle. When multiple beans match an injection point, `@Primary` can establish a default while `@Qualifier` can explicitly select a specific bean. `@Component` is useful for classes we control, while `@Bean` is useful when we need explicit construction logic or need to register third-party classes. `ApplicationContext` is the commonly used higher-level Spring container and extends the core capabilities of `BeanFactory`.

---

# 144\. Final Mental Picture

Remember this diagram:

```
                       SPRING
                         |
                         v
              +---------------------+
              | ApplicationContext  |
              +----------+----------+
                         |
          +--------------+--------------+
          |              |              |
          v              v              v
   Component Scan    @Configuration    @Bean
          |              |              |
          +--------------+--------------+
                         |
                         v
                  BeanDefinitions
                         |
                         v
                Dependency Resolution
                         |
              +----------+----------+
              |                     |
              v                     v
        Constructor DI          Qualifiers
              |                     |
              v                     v
       Fully Wired Beans      @Primary
              |
              v
       Lifecycle Management
              |
              v
       Application Usage
```

And the most important architectural transformation is:

```
BEFORE SPRING

OrderService
     |
     | new
     v
PaymentService

MANUAL DI

Main
 |
 +---- new PaymentService()
 |
 +---- new OrderService(payment)

SPRING DI

                Spring Container
                       |
            +----------+----------+
            |                     |
            v                     v
     PaymentService          OrderService
                                  |
                                  |
                         injected dependency
                                  |
                                  v
                          PaymentService
```

The fundamental idea is:

> **Don't make a class responsible for creating the objects it depends on. Let an external mechanism provide those dependencies. Spring's IoC container automates this object creation, wiring, and lifecycle management at application scale.**