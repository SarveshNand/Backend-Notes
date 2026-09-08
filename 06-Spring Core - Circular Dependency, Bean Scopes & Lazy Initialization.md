# Spring Core - Circular Dependency, Bean Scopes & Lazy Initialization

> **Topics covered**
>
>  1. Circular Dependency
> 2. Constructor vs Setter/Field Injection
> 3. Spring Boot Circular Reference Policy
> 4. Refactoring Circular Dependencies
> 5. Bean Scopes
> 6. Spring Singleton vs GoF Singleton
> 7. Prototype Scope & Lifecycle
> 8. Stateful vs Stateless Beans
> 9. Web Scopes
> 10. Eager vs Lazy Initialization
> 11. `@Lazy` and Proxies
> 12. Lazy Constructor Injection
> 13. Interview Questions & Coding Problems
> 14. Common Traps and Misconceptions

---

# 1\. Circular Dependency

## 1.1 What is a Circular Dependency?

A **circular dependency** occurs when two or more objects depend on each other directly or indirectly.

The simplest example is:

```
A → B
↑   ↓
└───┘
```

Meaning:

```
A needs B
B needs A
```

For example:

```
class OrderService {
    private final PaymentService paymentService;

    public OrderService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }
}
```

And:

```
class PaymentService {
    private final OrderService orderService;

    public PaymentService(OrderService orderService) {
        this.orderService = orderService;
    }
}
```

The dependency graph becomes:

```
┌─────────────────┐
│  OrderService   │
└────────┬────────┘
         │
         │ requires
         ▼
┌─────────────────┐
│ PaymentService  │
└────────┬────────┘
         │
         │ requires
         ▼
┌─────────────────┐
│  OrderService   │
└─────────────────┘

          ↑
          │
          └──────── Circular
```

There is no natural starting point for constructor creation.

---

# 2\. Linear Dependency vs Circular Dependency

Understanding a normal dependency chain makes circular dependency much easier to understand.

## 2.1 Linear Dependency

Suppose:

```
OrderService
     │
     ▼
PaymentService
     │
     ▼
PaymentGateway
```

In terms of "requires":

```
OrderService requires PaymentService

PaymentService requires PaymentGateway

PaymentGateway requires nothing
```

Therefore Spring can construct them from the bottom upward:

```
Step 1
PaymentGateway
     ↓
Step 2
PaymentService
     ↓
Step 3
OrderService
```

Conceptually:

```
┌───────────────────┐
│  PaymentGateway   │
└─────────┬─────────┘
          │
          ▼
┌───────────────────┐
│  PaymentService   │
└─────────┬─────────┘
          │
          ▼
┌───────────────────┐
│   OrderService    │
└───────────────────┘
```

This is easy for Spring.

---

# 3\. Circular Dependency with Constructor Injection

Now consider:

```
OrderService → PaymentService
PaymentService → OrderService
```

Suppose Spring starts with:

```
OrderService orderService =
        new OrderService(paymentService);
```

But it doesn't have `paymentService`.

So it attempts to create:

```
PaymentService paymentService =
        new PaymentService(orderService);
```

But it doesn't have `orderService`.

So it attempts to create:

```
OrderService orderService =
        new OrderService(paymentService);
```

And the cycle repeats.

```
Create OrderService
       │
       ▼
Need PaymentService
       │
       ▼
Create PaymentService
       │
       ▼
Need OrderService
       │
       ▼
Create OrderService
       │
       ▼
Need PaymentService
       │
       ▼
       ...
```

This is fundamentally impossible to complete using ordinary constructor creation.

---

# 4\. Pure Java Demonstration

This problem exists even without Spring.

Consider:

```
public class A {

    private B b;

    public A() {
        System.out.println("A created");
        b = new B();
    }
}
```

And:

```
public class B {

    private A a;

    public B() {
        System.out.println("B created");
        a = new A();
    }
}
```

Now:

```
public class Main {

    public static void main(String[] args) {
        A a = new A();
    }
}
```

Execution:

```
new A()
   │
   ▼
A constructor
   │
   ▼
new B()
   │
   ▼
B constructor
   │
   ▼
new A()
   │
   ▼
A constructor
   │
   ▼
new B()
   │
   ▼
B constructor
   │
   ▼
...
```

Eventually:

```
StackOverflowError
```

Why?

Every constructor call adds another stack frame.

```
┌──────────────────────┐
│ A constructor        │
├──────────────────────┤
│ B constructor        │
├──────────────────────┤
│ A constructor        │
├──────────────────────┤
│ B constructor        │
├──────────────────────┤
│ A constructor        │
├──────────────────────┤
│ B constructor        │
├──────────────────────┤
│ ...                  │
└──────────────────────┘
```

The call stack eventually runs out of space.

---

# 5\. Constructor Injection and Spring

Consider:

```
@Component
public class OrderService {

    private final PaymentService paymentService;

    public OrderService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }
}
```

And:

```
@Component
public class PaymentService {

    private final OrderService orderService;

    public PaymentService(OrderService orderService) {
        this.orderService = orderService;
    }
}
```

Spring sees:

```
OrderService
      │
      ▼
PaymentService
      │
      ▼
OrderService
      │
      ▼
...
```

Spring cannot finish creating either constructor because each constructor requires the other bean to already exist.

The result is typically:

```
BeanCurrentlyInCreationException
```

The important interview point is:

> Constructor-based circular dependencies cannot normally be resolved because neither object can be constructed without the other.

---

# 6\. Why Constructor Injection Exposes Design Problems

Constructor injection has an important advantage:

**Dependencies must exist before the object can exist.**

For example:

```
public OrderService(PaymentService paymentService) {
    this.paymentService = paymentService;
}
```

The following object is impossible:

```
OrderService
without PaymentService
```

This makes constructor injection very good at exposing dependency problems early.

That's one reason constructor injection is generally preferred.

---

# 7\. Setter/Field Injection and Circular Dependencies

Historically, Spring can resolve certain circular dependencies involving setter/field injection because object construction and dependency injection can happen in separate phases.

For example:

```
@Component
public class OrderService {

    @Autowired
    private PaymentService paymentService;
}
```

And:

```
@Component
public class PaymentService {

    @Autowired
    private OrderService orderService;
}
```

Conceptually:

```
Phase 1:
Create OrderService

Phase 2:
Create PaymentService

Phase 3:
Inject OrderService into PaymentService

Phase 4:
Inject PaymentService into OrderService
```

Diagram:

```
       ┌────────────────────┐
       │ Create OrderService│
       └──────────┬─────────┘
                  │
                  ▼
       ┌─────────────────────┐
       │ Create PaymentService│
       └──────────┬──────────┘
                  │
                  ▼
       ┌─────────────────────┐
       │ Inject OrderService │
       │ into PaymentService │
       └──────────┬──────────┘
                  │
                  ▼
       ┌──────────────────────┐
       │ Inject PaymentService│
       │ into OrderService    │
       └──────────────────────┘
```

The key difference is:

```
Constructor Injection

Create object
     +
Dependencies
     ↓
Complete object
```

versus:

```
Setter/Field Injection

Create object
     ↓
Incomplete object
     ↓
Inject dependencies
     ↓
Complete object
```

---

# 8\. Important Spring Boot Circular Dependency Rule

A very important interview fact:

**Spring Framework and Spring Boot are not exactly the same thing.**

Spring Framework has mechanisms for dealing with certain circular references.

Spring Boot 2.6 changed the default policy so that circular references are **not allowed by default**.

The property is:

```
spring.main.allow-circular-references=false
```

You can technically enable it:

```
spring.main.allow-circular-references=true
```

But this should generally be treated as a workaround rather than a design solution.

The preferred solution is:

```
Don't bypass the problem.
Refactor the dependency graph.
```

---

# 9\. Best Solution: Refactor the Design

Suppose we have:

```
OrderService
      ↓
PaymentService
      ↓
OrderService
```

Ask:

> Why does PaymentService need OrderService?

Maybe the code is:

```
public void pay() {
    System.out.println("Payment Done");

    orderService.getOrderDetails();
}
```

This is suspicious.

Payment processing shouldn't necessarily be responsible for obtaining or printing order details.

Instead:

```
OrderService
      │
      ├──────> PaymentService
      │
      └──────> Order Details
```

Example:

```
@Component
public class PaymentService {

    public void pay() {
        System.out.println("Payment Done");
    }
}
```

And:

```
@Component
public class OrderService {

    private final PaymentService paymentService;

    public OrderService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }

    public void placeOrder() {

        paymentService.pay();

        getOrderDetails();

        System.out.println("Order Placed");
    }

    public void getOrderDetails() {
        System.out.println("Order Details");
    }
}
```

Now:

```
OrderService
      │
      ▼
PaymentService
```

No cycle.

---

# 10\. An Even Better Architecture

Sometimes the real problem is that two services are trying to coordinate a business workflow.

Instead of:

```
OrderService ←→ PaymentService
```

introduce a coordinator:

```
                 ┌─────────────────┐
                 │ CheckoutService  │
                 └───────┬─────────┘
                         │
             ┌───────────┴───────────┐
             ▼                       ▼
      ┌──────────────┐       ┌───────────────┐
      │ OrderService │       │ PaymentService│
      └──────────────┘       └───────────────┘
```

For example:

```
@Component
public class CheckoutService {

    private final OrderService orderService;
    private final PaymentService paymentService;

    public CheckoutService(
            OrderService orderService,
            PaymentService paymentService) {

        this.orderService = orderService;
        this.paymentService = paymentService;
    }

    public void checkout() {

        orderService.validateOrder();

        paymentService.pay();

        orderService.completeOrder();
    }
}
```

Now:

```
CheckoutService
   │        │
   ▼        ▼
Order    Payment
```

The services don't need to know about each other.

This is often a much cleaner design.

---

# 11\. Bean Scope

A **bean scope** determines how Spring manages the lifetime and number of instances of a bean.

Think of:

```
"How many objects should Spring create?"
```

Spring's common scopes include:

```
Singleton
Prototype
Request
Session
Application
WebSocket
```

For basic Spring Core applications, the most important ones are:

```
Singleton
Prototype
```

---

# 12\. Singleton Scope

Singleton is the **default Spring bean scope**.

Example:

```
@Component
public class PaymentService {
}
```

Equivalent conceptually to:

```
@Component
@Scope("singleton")
public class PaymentService {
}
```

If we execute:

```
PaymentService p1 =
        context.getBean(PaymentService.class);

PaymentService p2 =
        context.getBean(PaymentService.class);
```

Then:

```
p1 == p2
```

is:

```
true
```

Diagram:

```
             ┌────────────────────┐
getBean() ──>│                    │
             │ PaymentService     │
getBean() ──>│  ONE INSTANCE      │
             │                    │
Class A ────>│                    │
Class B ────>│                    │
             └────────────────────┘
```

One Spring bean definition generally maps to one singleton instance within that application context.

---

# 13\. Important: Spring Singleton ≠ GoF Singleton

This is one of the most frequently asked interview questions.

## GoF Singleton

The classic Singleton Design Pattern attempts to ensure that only one instance exists within the relevant JVM/classloader context.

Example:

```
public class Singleton {

    private static Singleton instance;

    private Singleton() {
    }

    public static Singleton getInstance() {

        if (instance == null) {
            instance = new Singleton();
        }

        return instance;
    }
}
```

The class itself controls object creation.

---

## Spring Singleton

Spring does not fundamentally prevent you from writing:

```
PaymentService p =
        new PaymentService();
```

You can still create another object manually.

Spring's rule is:

```
One instance
per bean definition
per ApplicationContext
```

Therefore:

```
GoF Singleton
     ↓
Class-level restriction

Spring Singleton
     ↓
Container-level scope
```

---

# 14\. Multiple Spring ApplicationContexts

Suppose:

```
ApplicationContext A
       │
       └── PaymentService instance #1

ApplicationContext B
       │
       └── PaymentService instance #2
```

The two contexts can have different singleton instances.

Therefore the statement:

> "Spring creates exactly one object for the entire JVM."

is incorrect.

The better statement is:

> A Spring singleton is one instance per bean definition within a particular Spring IoC container/ApplicationContext.

---

# 15\. Prototype Scope

Prototype means:

> Create a new instance whenever the container is asked for the bean.

Example:

```
@Component
@Scope("prototype")
public class ShoppingCart {
}
```

Now:

```
ShoppingCart c1 =
        context.getBean(ShoppingCart.class);

ShoppingCart c2 =
        context.getBean(ShoppingCart.class);

ShoppingCart c3 =
        context.getBean(ShoppingCart.class);
```

Conceptually:

```
getBean()
   │
   ▼
Cart #1

getBean()
   │
   ▼
Cart #2

getBean()
   │
   ▼
Cart #3
```

Therefore:

```
c1 == c2
```

is:

```
false
```

---

# 16\. Singleton vs Prototype

```
             BEAN SCOPE
                 │
        ┌────────┴────────┐
        ▼                 ▼
    Singleton          Prototype
        │                 │
    One instance      New instance
    per context       per request
    of bean           to container
```

Example:

```
@Component
public class PaymentService {
}
```

versus:

```
@Component
@Scope("prototype")
public class ShoppingCart {
}
```

---

# 17\. Prototype and Dependency Injection — Important Interview Trap

Consider:

```
@Component
@Scope("prototype")
public class PrototypeBean {
}
```

And:

```
@Component
public class SingletonBean {

    private final PrototypeBean prototypeBean;

    public SingletonBean(PrototypeBean prototypeBean) {
        this.prototypeBean = prototypeBean;
    }
}
```

A common misconception is:

> Every time I call a method on SingletonBean, Spring gives me a new PrototypeBean.

That is **not** what happens.

The prototype dependency is normally created when the singleton is created and injected into it.

So:

```
SingletonBean
      │
      ▼
PrototypeBean #1
```

Calling:

```
singleton.doSomething();
singleton.doSomething();
singleton.doSomething();
```

does not automatically create:

```
PrototypeBean #1
PrototypeBean #2
PrototypeBean #3
```

The injected reference remains the same.

---

# 18\. How to Obtain a New Prototype from a Singleton

If a singleton needs a fresh prototype instance repeatedly, you need an appropriate lookup mechanism.

One option is:

```
ObjectProvider<PrototypeBean>
```

Example:

```
@Component
public class SingletonBean {

    private final ObjectProvider<PrototypeBean> provider;

    public SingletonBean(ObjectProvider<PrototypeBean> provider) {
        this.provider = provider;
    }

    public void execute() {

        PrototypeBean bean =
                provider.getObject();

        // Fresh prototype instance
    }
}
```

Conceptually:

```
SingletonBean
      │
      ▼
ObjectProvider
      │
      ├── getObject() → Prototype #1
      ├── getObject() → Prototype #2
      └── getObject() → Prototype #3
```

This is an excellent interview topic.

---

# 19\. Lifecycle of Singleton Beans

Spring has extensive lifecycle management for singleton beans.

Conceptually:

```
Bean Definition
      ↓
Instantiation
      ↓
Dependency Injection
      ↓
Initialization
      ↓
Bean Ready
      ↓
Application Running
      ↓
Context Shutdown
      ↓
Destruction callbacks
```

Spring can invoke destruction callbacks such as:

```
@PreDestroy
public void cleanup() {
    System.out.println("Cleaning up...");
}
```

---

# 20\. Prototype Bean Lifecycle

Prototype beans are different.

Spring:

```
Create
  ↓
Configure
  ↓
Inject dependencies
  ↓
Initialize
  ↓
Give object to caller
```

After handing it over, Spring generally does **not** manage its complete destruction lifecycle.

Diagram:

```
Spring
  │
  ├── Create prototype
  ├── Configure
  ├── Initialize
  │
  └── Give to application
           │
           ▼
       Application
           │
           ▼
       JVM manages
       object lifetime
```

Therefore, prototype beans should not be thought of as having the same destruction management as singleton beans.

---

# 21\. Stateful vs Stateless Beans

This is an important architecture question.

## Stateless Bean

A stateless service doesn't maintain request-specific mutable state in instance fields.

Example:

```
@Component
public class PaymentService {

    public void pay(double amount) {
        System.out.println("Paid: " + amount);
    }
}
```

The method uses:

```
amount
```

as a local/request parameter rather than storing it in the bean.

Such services are good candidates for singleton scope.

---

# 22\. Stateful Bean

A stateful object maintains data specific to a particular interaction/entity.

Example:

```
public class ShoppingCart {

    private List<String> products =
            new ArrayList<>();

    public void add(String product) {
        products.add(product);
    }
}
```

Different users may need different cart state:

```
User A
  ↓
Cart A
  ├── Laptop
  └── Mouse

User B
  ↓
Cart B
  ├── Keyboard
  └── Monitor
```

You must carefully choose how such state is scoped and managed.

---

# 23\. Why Services Are Usually Singleton

A typical Spring application looks like:

```
Controller
    │
    ▼
Service
    │
    ▼
Repository
    │
    ▼
Database
```

Controllers, services, and repositories are generally designed to be stateless.

Therefore singleton is normally appropriate.

For example:

```
@Service
public class OrderService {

    public void placeOrder(Long orderId) {
        // Business logic
    }
}
```

You generally don't want:

```
private Long currentUserId;
```

inside a singleton service.

That can create thread-safety and data-leakage problems.

---

# 24\. Singleton and Thread Safety

A very common interview question:

> "Is a Spring singleton thread-safe?"

**No.**

Singleton scope only means:

```
one instance per bean definition per context
```

It does **not** mean:

```
thread-safe
```

Suppose:

```
@Component
public class OrderService {

    private String currentOrder;

    public void process(String order) {
        currentOrder = order;

        // Processing
    }
}
```

If multiple requests access the same singleton:

```
Thread 1 ───────┐
                │
Thread 2 ───────┼──> SAME OrderService
                │
Thread 3 ───────┘
```

They can interfere with shared mutable state.

Better:

```
public void process(String order) {

    String currentOrder = order;

    // local variable
}
```

---

# 25\. Web Scopes

Spring applications that operate in a web environment provide additional scopes.

## Request Scope

```
HTTP Request #1
      ↓
RequestBean #1

HTTP Request #2
      ↓
RequestBean #2

HTTP Request #3
      ↓
RequestBean #3
```

Each HTTP request receives its own instance.

Annotation:

```
@RequestScope
public class RequestData {
}
```

---

# 26\. Session Scope

Session scope creates an instance for an HTTP session.

```
User A
   │
   └── Session
          │
          └── SessionBean A

User B
   │
   └── Session
          │
          └── SessionBean B
```

Example:

```
@SessionScope
@Component
public class UserSession {
}
```

The object can survive across multiple HTTP requests within the same session.

---

# 27\. Application Scope

Application scope is associated with the web application's `ServletContext`.

Conceptually:

```
Web Application
       │
       ▼
Application Scope
       │
       ▼
Application Bean
```

It is related to the lifecycle of the web application's servlet context.

---

# 28\. Scope Summary

| Scope | Instance Creation |
| --- | --- |
| Singleton | One per bean definition/context |
| Prototype | New instance when requested |
| Request | One per HTTP request |
| Session | One per HTTP session |
| Application | One per web application/ServletContext |
| WebSocket | One per WebSocket session |

---

# 29\. Bean Initialization

Bean scope answers:

> **How many instances?**

Initialization answers:

> **When is the instance created?**

These are different concepts.

```
Bean Configuration
       │
       ├── Scope → How many?
       │
       └── Initialization → When?
```

---

# 30\. Eager Initialization

Singleton beans are normally initialized eagerly by the application context.

Example:

```
@Component
public class PaymentService {

    public PaymentService() {
        System.out.println("PaymentService created");
    }
}
```

When the context starts:

```
ApplicationContext context =
        new AnnotationConfigApplicationContext(
                AppConfig.class);
```

Spring may create the singleton during context startup even before:

```
context.getBean(PaymentService.class);
```

is explicitly called.

---

# 31\. Why Eager Initialization?

One major reason is:

## Fail Fast

Suppose:

```
Application starts
      ↓
Spring creates beans
      ↓
Dependency problem
      ↓
Application fails immediately
```

This is useful because configuration errors are detected at startup rather than during a later request.

For production applications, discovering:

```
"DatabaseService cannot be created"
```

during startup is generally preferable to discovering it after the first user sends a request.

---

# 32\. Lazy Initialization

Lazy initialization means:

> Don't create the bean until it is actually needed.

Example:

```
@Component
@Lazy
public class PaymentService {

    public PaymentService() {
        System.out.println("PaymentService created");
    }
}
```

Now Spring can register the bean definition without immediately creating the object.

Later:

```
context.getBean(PaymentService.class);
```

causes creation.

---

# 33\. Eager vs Lazy

```
                 Bean
                  │
          ┌───────┴────────┐
          ▼                ▼
       Eager              Lazy
          │                │
    Create at startup   Create later
          │                │
       Fail fast        On demand
```

---

# 34\. Prototype Beans and Initialization

Prototype beans are not eagerly instantiated simply because their bean definition exists.

For example:

```
@Component
@Scope("prototype")
public class ReportGenerator {

    public ReportGenerator() {
        System.out.println("Created");
    }
}
```

Creating the context alone does not mean Spring creates every possible prototype instance.

A prototype instance is created when the container is asked to provide one or otherwise needs to resolve one.

This is why prototype scope is naturally associated with on-demand instance creation.

---

# 35\. `@Lazy` at Class Level

Example:

```
@Component
@Lazy
public class ExpensiveService {

    public ExpensiveService() {
        System.out.println("ExpensiveService created");
    }
}
```

Conceptually:

```
Application startup
       │
       ▼
Register ExpensiveService
       │
       X
   Don't create
       │
       ▼
Application running
       │
       ▼
Someone requests bean
       │
       ▼
Create ExpensiveService
```

---

# 36\. Important: `@Lazy` on a Dependency

This is particularly useful for circular dependencies.

Consider:

```
@Component
public class OrderService {

    private final PaymentService paymentService;

    public OrderService(@Lazy PaymentService paymentService) {
        this.paymentService = paymentService;
    }
}
```

Here `@Lazy` is applied to the **injection point**, not necessarily the entire class definition.

Conceptually Spring can inject a lazy-resolution proxy/reference.

```
OrderService
      │
      ▼
┌──────────────┐
│ Lazy Proxy   │
└──────┬───────┘
       │
       │ first actual use
       ▼
┌─────────────────┐
│ PaymentService   │
│ real instance    │
└─────────────────┘
```

---

# 37\. Why Does a Proxy Help?

Suppose:

```
A → B
B → A
```

Constructor creation normally requires:

```
Create A
 ↓
Need B
 ↓
Create B
 ↓
Need A
 ↓
Problem
```

But if A receives a lazy proxy for B:

```
Create A
 ↓
Need B
 ↓
Receive proxy for B
 ↓
A successfully constructed
```

Then:

```
Create B
 ↓
B receives A
 ↓
B successfully constructed
```

Later:

```
A
 ↓
proxy
 ↓
real B
```

The proxy acts as an intermediate reference that postpones the creation/resolution of the actual dependency.

---

# 38\. Constructor Circular Dependency + `@Lazy`

Example:

```
@Component
public class OrderService {

    private final PaymentService paymentService;

    public OrderService(
            @Lazy PaymentService paymentService) {

        this.paymentService = paymentService;
    }
}
```

And:

```
@Component
public class PaymentService {

    private final OrderService orderService;

    public PaymentService(OrderService orderService) {
        this.orderService = orderService;
    }
}
```

Dependency graph:

```
OrderService
     │
     │ lazy
     ▼
PaymentService
     │
     ▼
OrderService
```

The lazy edge can break the immediate construction cycle.

---

# 39\. Important Caveat About `@Lazy`

Although `@Lazy` can technically solve a circular dependency, don't immediately conclude:

> "Circular dependency is okay if I add `@Lazy`."

That's usually the wrong architectural lesson.

Prefer:

```
First choice:
Refactor dependency graph

Second choice:
Use lazy resolution only when there is a legitimate reason
```

For example:

```
Bad:

OrderService ←→ PaymentService

Better:

CheckoutService
     ├── OrderService
     └── PaymentService
```

---

# 40\. Global Lazy Initialization in Spring Boot

Spring Boot can globally enable lazy initialization:

```
spring.main.lazy-initialization=true
```

This changes the default initialization behavior so beans are generally initialized lazily unless otherwise configured.

Conceptually:

```
Default

Singleton
   ↓
Eager

Global Lazy

Singleton
   ↓
Lazy
```

You can override behavior for individual beans as appropriate.

---

# 41\. Why Not Make Everything Lazy?

Lazy initialization sounds attractive:

```
Faster startup!
```

But there are tradeoffs.

With eager initialization:

```
Startup
   ↓
Detect problems
```

With lazy initialization:

```
Startup
   ↓
Application appears healthy
   ↓
First request
   ↓
Create bean
   ↓
Configuration failure
```

So lazy initialization can move failures from startup to runtime.

---

# 42\. Eager vs Lazy — Interview Comparison

| Feature | Eager | Lazy |
| --- | --- | --- |
| Creation | Startup | On demand |
| Startup time | Usually higher | Usually lower |
| First-use latency | Lower | Can be higher |
| Failure detection | Earlier | Potentially later |
| Memory at startup | Usually higher | Usually lower |
| Good for | Critical infrastructure | Expensive/rarely used components |

---

# 43\. Circular Dependency — Interview Question #1

### Question

What is circular dependency?

### Answer

A circular dependency occurs when two or more beans directly or indirectly depend on each other.

Example:

```
A → B
↑   ↓
└───┘
```

With constructor injection:

```
A(B b)
B(A a)
```

Spring cannot normally instantiate either bean because each constructor requires the other bean first.

---

# 44\. Circular Dependency — Interview Question #2

### Question

Why does constructor injection make circular dependencies fail?

### Answer

Because constructor injection requires all mandatory dependencies to be available before the object can be constructed.

For:

```
A(B b)
B(A a)
```

Spring must create B before creating A, but B requires A. Therefore the container cannot complete the construction process.

---

# 45\. Circular Dependency — Interview Question #3

### Question

Can setter injection resolve circular dependencies?

### Answer

Spring has historically been able to resolve certain circular references involving setter/field injection because it can instantiate objects before completing dependency injection.

However, modern Spring Boot applications reject circular references by default.

The preferred solution is to refactor the design rather than depend on circular-reference support.

---

# 46\. Circular Dependency — Interview Question #4

### Question

How can `@Lazy` help with a circular dependency?

### Answer

`@Lazy` can defer resolution of a dependency. At an injection point, Spring can use a proxy/lazy reference instead of immediately requiring the real bean.

Example:

```
public OrderService(
        @Lazy PaymentService paymentService) {
    this.paymentService = paymentService;
}
```

This can break the immediate constructor-creation cycle.

---

# 47\. Circular Dependency — Interview Question #5

### Question

What is the best solution to circular dependency?

### Answer

**Refactor the design.**

For example, if:

```
OrderService ←→ PaymentService
```

exists because both services are coordinating a workflow, introduce a higher-level service:

```
          CheckoutService
            /         \
           /           \
          ▼             ▼
 OrderService      PaymentService
```

This produces a cleaner dependency graph.

---

# 48\. Bean Scope — Interview Question #1

### Question

What is the default scope of a Spring bean?

### Answer

```
Singleton
```

Example:

```
@Component
public class PaymentService {
}
```

is singleton-scoped unless another scope is configured.

---

# 49\. Bean Scope — Interview Question #2

### Question

Does Spring Singleton mean only one object can exist in the JVM?

### Answer

No.

Spring singleton means one instance for a particular bean definition within a particular Spring IoC container/ApplicationContext.

You can still:

```
new PaymentService();
```

outside Spring.

You can also have separate contexts with separate singleton instances.

---

# 50\. Bean Scope — Interview Question #3

### Question

Is a Spring singleton thread-safe?

### Answer

No.

Singleton controls **instance count**, not concurrency.

If a singleton contains mutable shared state:

```
private String currentUser;
```

multiple threads can access the same field concurrently.

Singleton beans should generally be designed to be stateless or otherwise made thread-safe.

---

# 51\. Bean Scope — Interview Question #4

### Question

When is Prototype scope useful?

### Answer

Prototype is useful when the application needs a new object instance whenever the bean is requested.

Typical examples include objects that represent temporary, stateful work.

However, simply saying "all DTOs should be prototype beans" is not correct. Many DTOs don't need to be Spring-managed beans at all.

---

# 52\. Bean Scope — Interview Question #5

### Question

What happens if a singleton depends directly on a prototype?

Consider:

```
@Component
@Scope("prototype")
class PrototypeBean {
}
```

and:

```
@Component
class SingletonBean {

    private final PrototypeBean prototypeBean;

    SingletonBean(PrototypeBean prototypeBean) {
        this.prototypeBean = prototypeBean;
    }
}
```

### Answer

A prototype instance is resolved when the singleton is created and injected into the singleton.

The singleton does **not** automatically receive a new prototype instance every time one of its methods executes.

For repeated fresh instances, use a lookup mechanism such as `ObjectProvider`.

---

# 53\. Bean Scope — Interview Question #6

### Question

What is the difference between Spring Singleton and GoF Singleton?

### Answer

```
GoF Singleton
→ Class/design-pattern level

Spring Singleton
→ Container/bean-scope level
```

GoF Singleton controls how the class itself creates instances.

Spring Singleton controls how the Spring container manages a particular bean definition.

---

# 54\. Lazy Initialization — Interview Question #1

### Question

What is lazy initialization?

### Answer

Lazy initialization means that a bean is not instantiated until it is actually required.

Example:

```
@Component
@Lazy
public class HeavyService {
}
```

Spring registers its definition but delays actual instantiation.

---

# 55\. Lazy Initialization — Interview Question #2

### Question

Are prototype beans eagerly initialized?

### Answer

No.

Prototype beans are created when instances are requested/needed rather than all being instantiated at application-context startup.

---

# 56\. Lazy Initialization — Interview Question #3

### Question

What happens if an eager bean depends on a lazy bean?

Consider:

```
@Component
@Lazy
class PaymentService {
}
```

and:

```
@Component
class OrderService {

    OrderService(PaymentService paymentService) {
    }
}
```

The fact that `PaymentService` itself is lazy does not mean it can never be created during startup.

If eager `OrderService` needs the actual `PaymentService` instance while it is being constructed, Spring may have to resolve/create that dependency.

To defer that dependency, use lazy injection:

```
OrderService(@Lazy PaymentService paymentService)
```

This distinction is highly interview-worthy.

---

# 57\. Lazy Initialization — Interview Question #4

### Question

Why use `@Lazy`?

Possible reasons include:

- Expensive bean creation
- Rarely used functionality
- Reducing startup work
- Breaking certain dependency initialization cycles
- Deferring external-resource initialization

But it should not be used as a blanket replacement for good architecture.

---

# 58\. Coding Interview Problem #1

## Identify the Problem

Given:

```
@Component
class A {

    private final B b;

    A(B b) {
        this.b = b;
    }
}
```

```
@Component
class B {

    private final A a;

    B(A a) {
        this.a = a;
    }
}
```

### Question

What happens?

### Answer

There is a constructor circular dependency:

```
A → B
↑   ↓
└───┘
```

Spring cannot normally construct these beans and will fail during context initialization.

---

# 59\. Coding Interview Problem #2

## Fix the Circular Dependency

Given:

```
@Component
class A {

    private final B b;

    A(B b) {
        this.b = b;
    }
}
```

```
@Component
class B {

    private final A a;

    B(A a) {
        this.a = a;
    }
}
```

### Possible Technical Fix

```
@Component
class A {

    private final B b;

    A(@Lazy B b) {
        this.b = b;
    }
}
```

This may allow Spring to defer B's resolution.

### Better Design Fix

Ask:

> Why does B need A?

If B doesn't actually need A, remove the dependency.

If both are coordinating a workflow:

```
A ←→ B
```

consider:

```
Coordinator
   ├── A
   └── B
```

---

# 60\. Coding Interview Problem #3

Consider:

```
@Component
@Scope("prototype")
class Counter {

    private int count = 0;

    public void increment() {
        count++;
    }

    public int getCount() {
        return count;
    }
}
```

And:

```
Counter c1 = context.getBean(Counter.class);
Counter c2 = context.getBean(Counter.class);

c1.increment();

System.out.println(c1.getCount());
System.out.println(c2.getCount());
```

### What is the output?

```
1
0
```

Because:

```
c1 → Counter #1
c2 → Counter #2
```

---

# 61\. Coding Interview Problem #4

Given:

```
@Component
class A {

    private final B b;

    A(B b) {
        this.b = b;
    }
}
```

```
@Component
@Scope("prototype")
class B {
}
```

### Question

Will a new B be created every time A's method is called?

### Answer

No.

If A is singleton-scoped, the prototype dependency is normally created when A is instantiated and injected into A.

The dependency reference inside A doesn't automatically become a fresh object on every method invocation.

---

# 62\. Coding Interview Problem #5

How can we request a new prototype instance from a singleton?

```
@Component
class A {

    private final ObjectProvider<B> provider;

    A(ObjectProvider<B> provider) {
        this.provider = provider;
    }

    public void execute() {

        B b1 = provider.getObject();
        B b2 = provider.getObject();
    }
}
```

Now:

```
b1 → B #1
b2 → B #2
```

assuming B is prototype-scoped.

---

# 63\. Scenario-Based Interview Question

### Scenario

Your application has:

```
100+ services
```

The application startup takes:

```
30 seconds
```

Most services are rarely used.

What could you consider?

### Answer

Lazy initialization may reduce startup work:

```
spring.main.lazy-initialization=true
```

However, you should first identify why startup is slow.

Lazy initialization has a tradeoff:

```
Faster startup
       ↓
Potentially slower first use
       +
Some configuration failures discovered later
```

So it is a performance/design decision, not a universal optimization.

---

# 64\. Scenario-Based Interview Question

### Scenario

You have:

```
@Component
public class PaymentService {

    private String currentTransaction;
}
```

The bean is singleton-scoped.

100 HTTP requests arrive concurrently.

### Is this safe?

No.

All requests can access the same instance:

```
Request 1 ──┐
Request 2 ──┼──> PaymentService singleton
Request 3 ──┘
```

Therefore:

```
private String currentTransaction;
```

is shared mutable state.

A better design is to keep request-specific data in method-local variables or appropriate request/session-scoped components.

---

# 65\. Scenario-Based Interview Question

### Scenario

You have:

```
OrderService
      ↓
PaymentService
      ↓
OrderService
```

The developer says:

> "I'll just enable `allow-circular-references=true`."

Should you accept this solution?

Usually no.

Ask:

```
Why does PaymentService need OrderService?
```

If the dependency exists because responsibilities are mixed, refactor.

Possible result:

```
CheckoutService
    ├── OrderService
    └── PaymentService
```

---

# 66\. Common Interview Traps

## Trap 1

> Spring singleton means only one object can ever exist.

**Wrong.**

Correct:

```
One instance per bean definition per ApplicationContext.
```

---

## Trap 2

> Singleton means thread-safe.

**Wrong.**

Singleton says nothing about thread safety.

---

## Trap 3

> Prototype dependency inside singleton automatically creates a new object for every method call.

**Wrong.**

The dependency is normally resolved during singleton creation.

---

## Trap 4

> `@Lazy` always means the bean will never be created at startup.

**Wrong.**

Its actual creation depends on how and where it is injected/used.

---

## Trap 5

> Circular dependencies are a Spring bug.

**Wrong.**

They are fundamentally dependency-graph/design problems, although Spring's handling depends on injection style and configuration.

---

## Trap 6

> Field injection is better because it can solve circular dependencies.

Usually **wrong as a design conclusion**.

The ability to work around a cycle doesn't mean the cycle is good architecture.

---

# 67\. Dependency Graphs — Think Like an Interviewer

Whenever you see Spring dependency code, draw a graph.

Example:

```
A(B b)
B(C c)
C(D d)
D()
```

Draw:

```
A
↓
B
↓
C
↓
D
```

This is:

```
Linear Dependency
```

Spring can resolve it naturally.

Now:

```
A(B b)
B(C c)
C(A a)
```

Draw:

```
A
↓
B
↓
C
↓
A
```

Immediately identify:

```
CIRCULAR DEPENDENCY
```

This is one of the easiest ways to reason about Spring bean creation.

---

# 68\. Dependency Graph and Creation Order

For:

```
OrderService
      ↓
PaymentService
      ↓
PaymentGateway
```

Think:

```
PaymentGateway
       ↓
PaymentService
       ↓
OrderService
```

Creation proceeds from dependencies toward dependents.

Conceptually:

```
1. PaymentGateway
2. PaymentService
3. OrderService
```

---

# 69\. Circular Dependency Graph

For:

```
OrderService
      ↓
PaymentService
      ↓
OrderService
```

there is no valid bottom-level starting node.

```
       ┌───────────────┐
       │ OrderService  │
       └───────┬───────┘
               │
               ▼
       ┌───────────────┐
       │PaymentService │
       └───────┬───────┘
               │
               └───────────────┐
                               │
                               ▼
                       OrderService
```

This is the mental model you should use in interviews.

---

# 70\. Complete Scope Mental Model

Remember:

```
                 SPRING BEAN
                     │
          ┌──────────┴──────────┐
          │                     │
          ▼                     ▼
       SCOPE              INITIALIZATION
          │                     │
    ┌─────┴─────┐          ┌────┴────┐
    ▼           ▼          ▼         ▼
Singleton   Prototype    Eager      Lazy
    │           │
    │           │
One instance   New instance
per context    when requested
```

Scope and initialization answer different questions.

---

# 71\. Complete Circular Dependency Mental Model

```
          Dependency Graph
                 │
       ┌─────────┴─────────┐
       ▼                   ▼
    Linear              Circular
       │                   │
       ▼                   ▼
A → B → C              A → B
                       ↑   ↓
                       └───┘
       │                   │
       ▼                   ▼
Easy to resolve       Constructor injection
                      normally fails
                           │
                           ▼
                    Possible lazy proxy
                           │
                           ▼
                    Better: refactor
```

---

# 72\. Complete Lazy Initialization Mental Model

```
                 BEAN
                   │
          ┌────────┴────────┐
          ▼                 ▼
        Eager              Lazy
          │                 │
          ▼                 ▼
   Create at startup   Create on demand
          │                 │
          ▼                 ▼
    Fail fast          Deferred failure
```

---

# 73\. Quick Revision Table

| Topic | Key Point |
| --- | --- |
| Circular Dependency | A dependency graph contains a cycle |
| Constructor Circular Dependency | Normally cannot be resolved directly |
| `BeanCurrentlyInCreationException` | Common symptom of unresolvable circular creation |
| Setter/Field Injection | Can allow certain circular references |
| Spring Boot 2.6+ | Circular references disabled by default |
| Best Circular Dependency Solution | Refactor the architecture |
| `@Lazy` | Defers bean/dependency resolution |
| Lazy Injection | Can use a proxy/reference to defer actual creation |
| Singleton | Default scope |
| Prototype | New instance when requested |
| Spring Singleton | One per bean definition/context |
| GoF Singleton | Class/design-pattern restriction |
| Singleton Thread Safety | Not guaranteed |
| Prototype in Singleton | Doesn't automatically create a new prototype per method call |
| `ObjectProvider` | Useful for obtaining fresh prototype instances |
| Request Scope | One bean instance per HTTP request |
| Session Scope | One instance per HTTP session |
| Eager | Created during startup |
| Lazy | Created when needed |
| Global Lazy | `spring.main.lazy-initialization=true` |

---

# 74\. Interview Cheat Sheet

If the interviewer asks:

### "What is circular dependency?"

Say:

> A circular dependency occurs when two or more beans depend on each other directly or indirectly, creating a cycle in the dependency graph.

### "Why does constructor injection fail?"

> Because each constructor requires the other bean to already exist, so neither bean can be fully constructed first.

### "Can Spring resolve circular dependencies?"

> Certain circular references involving setter/field injection can historically be resolved by Spring, but modern Spring Boot disables circular references by default. Constructor-based cycles generally cannot be resolved directly.

### "How can `@Lazy` help?"

> It defers dependency resolution and can allow Spring to inject a lazy proxy, breaking the immediate constructor-creation cycle.

### "What's the best solution?"

> Refactor the dependency graph and remove the cycle rather than relying on circular-reference configuration.

### "What's the default bean scope?"

> Singleton.

### "Does Spring Singleton mean one object per JVM?"

> No. It means one instance per bean definition within a particular ApplicationContext.

### "Is singleton thread-safe?"

> No. Singleton is a scope, not a thread-safety guarantee.

### "What is prototype?"

> A new bean instance is created whenever the container is asked for one.

### "Does a singleton automatically get a new prototype every time?"

> No. A directly injected prototype is normally resolved when the singleton is created. Use a lookup mechanism such as `ObjectProvider` when fresh instances are required repeatedly.

### "What is lazy initialization?"

> The bean's actual instance creation is postponed until it is needed.

### "Why use eager initialization?"

> It follows a fail-fast model and detects many configuration and dependency problems during application startup.

---

# 75\. Final Mental Model

The most important thing to understand is that Spring is fundamentally managing an **object graph**.

Suppose your application contains:

```
                   ApplicationContext
                          │
             ┌────────────┼────────────┐
             │            │            │
             ▼            ▼            ▼
          Order       Payment       Gateway
             │            │
             └──────┬─────┘
                    │
                    ▼
                Dependencies
```

Spring has to answer four fundamental questions:

```
1. WHICH objects should I manage?
       ↓
@Component / @Bean

2. HOW should they depend on each other?
       ↓
@Autowired / constructor injection

3. HOW MANY instances should exist?
       ↓
@Scope

4. WHEN should they be created?
       ↓
Eager / @Lazy
```

And when the dependency graph becomes:

```
A → B → C → A
```

you have a circular dependency.

The best response isn't:

```
"How do I force Spring to accept this?"
```

Instead ask:

```
"Why does my architecture require this cycle?"
```

That mindset is much more valuable in a Spring interview than memorizing annotations.

---

# 76\. One-Page Revision

```
                    SPRING BEAN MANAGEMENT
                            │
            ┌───────────────┼────────────────┐
            │               │                │
            ▼               ▼                ▼
       DEPENDENCY         SCOPE         INITIALIZATION
            │               │                │
            ▼               ▼                ▼
       Constructor      Singleton          Eager
       Setter           Prototype           Lazy
       Field            Request
                        Session
                        Application
            │
            ▼
       Dependency Graph
            │
       ┌────┴─────┐
       ▼          ▼
    Linear      Circular
       │          │
       ▼          ▼
     Easy      Constructor
                fails
                  │
             ┌────┴─────┐
             ▼          ▼
          @Lazy      Refactor
          proxy      (preferred)
```

## Golden Rules

1. **Prefer constructor injection.**
2. **Keep services stateless whenever possible.**
3. **Don't confuse Spring Singleton with GoF Singleton.**
4. **Singleton does not mean thread-safe.**
5. **Prototype does not mean "new instance on every method call."**
6. **Use `ObjectProvider`/lookup mechanisms when a singleton needs fresh prototype instances.**
7. **Use `@Lazy` deliberately, not as a blanket fix.**
8. **Treat circular dependencies as an architectural smell.**
9. **Draw the dependency graph when debugging Spring bean creation.**
10. **Remember: Scope answers "how many?" while initialization answers "when?"**
