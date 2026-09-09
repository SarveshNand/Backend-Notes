# Spring Bean Lifecycle - Detailed Notes, Diagrams & Interview Questions

---

# 1\. What is the Spring Bean Lifecycle?

In normal Java, we are responsible for creating objects:

```
UserService userService = new UserService();
```

The JVM manages the object's memory and eventually Garbage Collection.

In Spring, the **IoC (Inversion of Control) Container** takes responsibility for managing Spring Beans.

Spring controls:

- Bean creation
- Dependency injection
- Bean initialization
- Lifecycle callbacks
- Bean destruction

The overall lifecycle can be visualized as:

```
                         SPRING BEAN LIFECYCLE
┌─────────────────────────────────────────────────────────────────────┐
│                                                                     │
│  1. Container Starts                                                │
│          │                                                          │
│          ▼                                                          │
│  2. Read Configuration                                              │
│          │                                                          │
│          ▼                                                          │
│  3. Create BeanDefinitions                                          │
│          │                                                          │
│          ▼                                                          │
│  4. Instantiate Bean                                                │
│          │                                                          │
│          ▼                                                          │
│  5. Dependency Injection                                            │
│          │                                                          │
│          ▼                                                          │
│  6. Aware Callbacks                                                 │
│          │                                                          │
│          ▼                                                          │
│  7. BeanPostProcessor - Before Initialization                       │
│          │                                                          │
│          ▼                                                          │
│  8. @PostConstruct / Initialization Callbacks                      │
│          │                                                          │
│          ▼                                                          │
│  9. BeanPostProcessor - After Initialization                        │
│          │                                                          │
│          ▼                                                          │
│  10. Bean Ready / Available                                         │
│          │                                                          │
│          │      Application runs                                    │
│          │                                                          │
│          ▼                                                          │
│  11. ApplicationContext.close()                                     │
│          │                                                          │
│          ▼                                                          │
│  12. Destruction Callbacks                                          │
│          │                                                          │
│          ▼                                                          │
│  13. Bean Destroyed / Eligible for GC                               │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

# 2\. What is an IoC Container?

IoC means **Inversion of Control**.

Normally:

```
PaymentService paymentService = new PaymentService();
OrderService orderService = new OrderService(paymentService);
```

Our code controls object creation.

With Spring:

```
ApplicationContext context =
        new AnnotationConfigApplicationContext(AppConfig.class);
```

Spring creates and wires the objects.

```
Without Spring
────────────────────────────────────────

Application
     │
     ├── new PaymentService()
     │
     └── new OrderService(paymentService)

With Spring
────────────────────────────────────────

                 Spring IoC Container
                         │
              ┌──────────┴──────────┐
              │                     │
              ▼                     ▼
       PaymentService          OrderService
              │                     │
              └──── dependency ─────┘
```

The developer declares **what is required**, while Spring determines **how and when objects should be created and connected**.

---

# 3\. Spring IoC Container Types

The two important container interfaces are:

```
BeanFactory
    │
    └── Basic IoC Container

ApplicationContext
    │
    └── Advanced IoC Container
```

Common implementation:

```
ApplicationContext context =
        new AnnotationConfigApplicationContext(AppConfig.class);
```

`ApplicationContext` provides additional features such as:

- Event handling
- Internationalization
- Resource loading
- Automatic BeanPostProcessor registration
- Integration with Spring's broader infrastructure

For most modern Spring applications, you will work with `ApplicationContext`.

---

# 4\. Project Setup

A minimal Maven project can use Spring Context.

```
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>6.x.x</version>
</dependency>
```

For `@PostConstruct` and `@PreDestroy`, use Jakarta annotations:

```
<dependency>
    <groupId>jakarta.annotation</groupId>
    <artifactId>jakarta.annotation-api</artifactId>
    <version>3.x.x</version>
</dependency>
```

> In Spring 6+, use `jakarta.annotation.*`, not `javax.annotation.*`.

Imports:

```
import jakarta.annotation.PostConstruct;
import jakarta.annotation.PreDestroy;
```

---

# 5\. Configuration Class

Example:

```
package in.coderarmy;

import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

@Configuration
@ComponentScan("in.coderarmy")
public class AppConfig {
}
```

We can start the container:

```
import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;

public class Main {

    public static void main(String[] args) {

        ApplicationContext context =
                new AnnotationConfigApplicationContext(AppConfig.class);

    }
}
```

---

# 6\. Bean Definition

Before Spring creates an actual object, it maintains metadata describing how the bean should be created.

This metadata is represented by a **BeanDefinition**.

Think of it as a blueprint.

```
                 BeanDefinition
                       │
        ┌──────────────┼──────────────┐
        │              │              │
        ▼              ▼              ▼
    Bean Name       Bean Class       Scope
        │              │              │
        ▼              ▼              ▼
 "orderService"   OrderService.class Singleton
                                      │
                                      ▼
                                   Lazy?
                                      │
                                      ▼
                                    false
```

A BeanDefinition may contain information such as:

```
Bean Name
Bean Class
Scope
Lazy configuration
Constructor information
Dependency information
Initialization method
Destruction method
Autowiring information
```

Example conceptually:

```
BeanDefinition: OrderService

--------------------------------------
Bean Name       : orderService
Class           : OrderService
Scope           : singleton
Lazy            : false
Init Method     : init
Destroy Method  : cleanup
Dependencies    : PaymentService
--------------------------------------
```

### Important Interview Point

Spring generally needs to know **how a bean should be created before actually creating the bean**.

Therefore:

```
Configuration
      ↓
Bean Definitions
      ↓
Bean Instances
```

Do not confuse:

```
BeanDefinition ≠ Bean Object
```

A BeanDefinition is metadata/blueprint.

A bean object is the actual Java instance.

---

# 7\. Complete Lifecycle Diagram

A more technically useful lifecycle diagram is:

```
                 APPLICATION CONTEXT STARTS
                            │
                            ▼
                 Read Configuration Metadata
                            │
                            ▼
                  Register BeanDefinitions
                            │
                            ▼
              ┌───────────────────────────────┐
              │ Is bean an eager singleton?   │
              └───────────────┬───────────────┘
                              │
                    ┌─────────┴─────────┐
                    │                   │
                   YES                  NO
                    │                   │
                    ▼                   ▼
              Create Bean         Wait until needed
                    │                   │
                    │             getBean() / injection
                    │                   │
                    └─────────┬─────────┘
                              ▼
                       Instantiate Bean
                              │
                              ▼
                    Dependency Injection
                              │
                              ▼
                       Aware Callbacks
                              │
                              ▼
                  BeanPostProcessor
                    Before Initialization
                              │
                              ▼
                     @PostConstruct
                              │
                              ▼
                  InitializingBean
                  afterPropertiesSet()
                              │
                              ▼
                    Custom initMethod
                              │
                              ▼
                  BeanPostProcessor
                     After Initialization
                              │
                              ▼
                        Bean Ready
                              │
                              ▼
                 Application Uses Bean
                              │
                              ▼
               ApplicationContext.close()
                              │
                              ▼
                    Destruction Phase
                              │
                              ▼
                      @PreDestroy
                              │
                              ▼
                    DisposableBean
                         destroy()
                              │
                              ▼
                     Custom destroyMethod
                              │
                              ▼
                       Bean Destroyed
```

---

# 8\. Step 1 — Container Startup

The lifecycle begins when we create the Spring container.

```
ApplicationContext context =
        new AnnotationConfigApplicationContext(AppConfig.class);
```

For shutdown:

```
ConfigurableApplicationContext context =
        new AnnotationConfigApplicationContext(AppConfig.class);

context.close();
```

Why use `ConfigurableApplicationContext`?

Because `ApplicationContext` itself does not expose `close()`.

```
ApplicationContext
        │
        │ does not directly expose close()
        ▼
ConfigurableApplicationContext
        │
        └── close()
```

---

# 9\. Step 2 — Configuration Processing

Spring reads configuration metadata.

For example:

```
@Configuration
@ComponentScan("in.coderarmy")
public class AppConfig {
}
```

Spring detects:

```
@Configuration
@ComponentScan
@Bean
@Component
@Service
@Repository
@Controller
```

and other configuration information.

Conceptually:

```
AppConfig.class
      │
      ▼
Spring Configuration Processing
      │
      ├── @Configuration
      ├── @ComponentScan
      ├── @Bean
      └── other metadata
```

---

# 10\. Step 3 — Component Scanning

Suppose we have:

```
@Service
public class OrderService {
}
```

and:

```
@Service
public class PaymentService {
}
```

With:

```
@ComponentScan("in.coderarmy")
```

Spring scans the specified package and discovers candidate components.

```
in.coderarmy
     │
     ├── AppConfig
     │
     ├── OrderService
     │
     └── PaymentService
```

Spring then registers bean definitions for the discovered beans.

---

# 11\. Step 4 — Bean Instantiation

Now Spring creates the actual Java object.

Example:

```
@Service
public class OrderService {

    public OrderService() {
        System.out.println("OrderService Constructor");
    }
}
```

Spring eventually creates:

```
new OrderService();
```

Conceptually, Spring uses its bean creation machinery and reflection/factory mechanisms to instantiate the object.

### Important Correction

Do not oversimplify this in an interview as:

> "Spring always directly uses Reflection to create every bean."

A better answer is:

> "Spring uses its bean creation infrastructure, which can use reflection and various instantiation strategies depending on the bean definition and configuration."

---

# 12\. Constructor Injection

Consider:

```
@Service
public class PaymentService {
}
```

```
@Service
public class OrderService {

    private final PaymentService paymentService;

    public OrderService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }
}
```

Spring sees:

```
OrderService
     │
     └── requires PaymentService
                 │
                 ▼
         Create PaymentService
                 │
                 ▼
         Create OrderService
         using PaymentService
```

Conceptually:

```
PaymentService paymentService =
        new PaymentService();

OrderService orderService =
        new OrderService(paymentService);
```

The actual Spring internals are considerably more sophisticated, but this is the correct conceptual model.

---

# 13\. Constructor Injection vs Field Injection

## Constructor Injection

```
@Service
public class OrderService {

    private final PaymentService paymentService;

    public OrderService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }
}
```

Conceptually:

```
Create dependencies
       ↓
Create object with dependencies
       ↓
Object is already wired
```

## Field Injection

```
@Service
public class OrderService {

    @Autowired
    private PaymentService paymentService;
}
```

Conceptually:

```
Create OrderService
       ↓
Inject PaymentService into field
       ↓
Continue lifecycle
```

### Interview Recommendation

Prefer constructor injection because it:

- Makes dependencies explicit
- Supports immutability
- Makes testing easier
- Helps detect mandatory dependencies
- Avoids partially initialized objects

---

# 14\. Step 5 — Dependency Injection

Spring can perform dependency injection through:

```
Dependency Injection
       │
       ├── Constructor Injection
       │
       ├── Setter Injection
       │
       └── Field Injection
```

Example:

```
@Service
public class OrderService {

    private PaymentService paymentService;

    @Autowired
    public void setPaymentService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }
}
```

The object is instantiated first and the dependency is then injected.

---

# 15\. Step 6 — Aware Interfaces

Spring provides several `Aware` interfaces.

Common examples:

```
BeanNameAware
ApplicationContextAware
BeanFactoryAware
EnvironmentAware
ResourceLoaderAware
```

These allow a bean to request certain container-related information.

---

# 16\. BeanNameAware

Example:

```
@Component
public class UserService implements BeanNameAware {

    @Override
    public void setBeanName(String name) {
        System.out.println(
            "Bean name = " + name
        );
    }
}
```

If the bean is registered as:

```
userService
```

Spring calls:

```
setBeanName("userService");
```

Conceptually:

```
Dependency Injection
        ↓
BeanNameAware
        ↓
setBeanName("userService")
```

---

# 17\. ApplicationContextAware

Example:

```
@Component
public class UserService
        implements ApplicationContextAware {

    @Override
    public void setApplicationContext(
            ApplicationContext applicationContext) {

        System.out.println(
            applicationContext.getClass().getName()
        );
    }
}
```

Spring provides the `ApplicationContext`.

```
UserService
     │
     ▼
ApplicationContextAware
     │
     ▼
setApplicationContext(context)
```

### Caution

Although useful for framework integration and special cases, directly depending on `ApplicationContext` can increase coupling.

Prefer normal dependency injection where possible.

---

# 18\. Step 7 — BeanPostProcessor

This is a **very important interview topic**.

Spring provides the `BeanPostProcessor` extension point.

It can execute logic:

```
Before Initialization
          │
          ▼
Initialization
          │
          ▼
After Initialization
```

Example:

```
@Component
public class MyBeanPostProcessor
        implements BeanPostProcessor {

    @Override
    public Object postProcessBeforeInitialization(
            Object bean,
            String beanName) {

        System.out.println(
            "Before Init: " + beanName
        );

        return bean;
    }

    @Override
    public Object postProcessAfterInitialization(
            Object bean,
            String beanName) {

        System.out.println(
            "After Init: " + beanName
        );

        return bean;
    }
}
```

---

# 19\. Important Lifecycle Order

A simplified lifecycle sequence is:

```
1. Constructor
       ↓
2. Dependency Injection
       ↓
3. Aware callbacks
       ↓
4. BeanPostProcessor.beforeInitialization()
       ↓
5. @PostConstruct
       ↓
6. InitializingBean.afterPropertiesSet()
       ↓
7. Custom init-method
       ↓
8. BeanPostProcessor.afterInitialization()
       ↓
9. Bean Ready
```

This is a highly useful interview sequence.

---

# 20\. Initialization Callback #1 — @PostConstruct

Modern applications commonly use:

```
@PostConstruct
public void init() {
    System.out.println("Bean initialized");
}
```

Example:

```
@Component
public class CartService {

    private final Map<Integer, String> cartMap =
            new HashMap<>();

    @PostConstruct
    public void init() {

        System.out.println(
            "Loading cart data..."
        );

        cartMap.put(1, "Laptop");
        cartMap.put(2, "Mouse");
    }
}
```

At this point dependencies have already been injected.

---

# 21\. Why @PostConstruct?

Consider:

```
@Component
public class OrderService {

    @Autowired
    private PaymentService paymentService;

    public OrderService() {

        // paymentService is not safe to use here
        // for field injection
    }

    @PostConstruct
    public void init() {

        // Dependency is available here
        paymentService.initialize();
    }
}
```

The constructor executes during object creation.

The `@PostConstruct` method runs later in the initialization lifecycle.

Therefore:

```
Constructor
     │
     │ object created
     ▼
Dependency Injection
     │
     ▼
@PostConstruct
     │
     │ dependencies available
     ▼
Bean Ready
```

---

# 22\. Constructor vs @PostConstruct

| Feature | Constructor | `@PostConstruct` |
| --- | --- | --- |
| Object creation | Yes | No |
| Runs immediately during construction | Yes | No |
| Field/setter dependencies guaranteed | No | Yes |
| Good for mandatory constructor dependencies | Yes | No |
| Good for post-injection initialization | No | Yes |
| Should generally be lightweight | Yes | Prefer reasonably controlled initialization |

### Important Point

Do not interpret "`@PostConstruct` means heavy initialization should always happen here."

Heavy work can delay application startup.

A better principle is:

> Use `@PostConstruct` for initialization that requires the bean's dependencies to already be available, and avoid unnecessarily expensive startup work.

---

# 23\. Initialization Callback #2 — InitializingBean

A bean can implement:

```
InitializingBean
```

Example:

```
@Component
public class PaymentService
        implements InitializingBean {

    @Override
    public void afterPropertiesSet() {

        System.out.println(
            "PaymentService initialized"
        );
    }
}
```

Spring calls:

```
afterPropertiesSet();
```

### Is it commonly preferred?

Usually, application code favors annotations such as:

```
@PostConstruct
```

because they avoid coupling the application class to a Spring-specific lifecycle interface.

---

# 24\. Initialization Callback #3 — initMethod

Suppose:

```
public class PaymentClient {

    public void start() {
        System.out.println("PaymentClient started");
    }
}
```

Configuration:

```
@Configuration
public class AppConfig {

    @Bean(initMethod = "start")
    public PaymentClient paymentClient() {
        return new PaymentClient();
    }
}
```

Spring calls:

```
paymentClient.start();
```

This can be useful when you cannot modify a third-party class.

---

# 25\. Initialization Methods — Comparison

```
                  INITIALIZATION
                        │
        ┌───────────────┼────────────────┐
        │               │                │
        ▼               ▼                ▼
 @PostConstruct   InitializingBean   initMethod
        │               │                │
        │               │                │
 Annotation       Spring interface    @Bean config
 based            based               based
```

### Typical preference

```
Application code
      ↓
@PostConstruct

Need Spring lifecycle interface
      ↓
InitializingBean

Third-party/custom external class
      ↓
initMethod
```

---

# 26\. Step 8 — BeanPostProcessor After Initialization

After initialization callbacks, Spring can invoke:

```
postProcessAfterInitialization()
```

This is important because Spring may return a different object from the original bean.

This mechanism is involved in various Spring features, including proxy-based behavior.

Conceptually:

```
Original Bean
     │
     ▼
BeanPostProcessor
     │
     ▼
Possibly wrapped/proxied object
     │
     ▼
Bean exposed to application
```

This concept becomes important when studying:

- Spring AOP
- `@Transactional`
- Security proxies
- Caching
- Other proxy-based infrastructure

---

# 27\. Step 9 — Bean Ready

After the relevant initialization processing is complete, the bean is ready to be used.

For singleton beans:

```
Bean created
     ↓
Bean initialized
     ↓
Singleton stored by container
     ↓
getBean() returns managed instance
```

Example:

```
OrderService service =
        context.getBean(OrderService.class);

service.placeOrder();
```

---

# 28\. Singleton Scope

By default, Spring beans are singleton scoped.

```
@Component
public class OrderService {
}
```

Spring normally creates one instance per Spring container.

```
ApplicationContext
       │
       └──── OrderService
                │
                │ one managed instance
                ▼
          OrderService@1234
```

Important:

> Spring singleton means one instance per Spring IoC container, not one instance for the entire JVM in every possible circumstance.

---

# 29\. Eager Singleton

By default, singleton beans are generally created eagerly during context startup.

```
@Component
public class OrderService {

    public OrderService() {
        System.out.println("Constructor");
    }

    @PostConstruct
    public void init() {
        System.out.println("Init");
    }
}
```

When the context starts:

```
ApplicationContext starts
       ↓
OrderService created
       ↓
Constructor
       ↓
Injection
       ↓
@PostConstruct
       ↓
Bean ready
```

---

# 30\. Lazy Singleton

Use:

```
@Lazy
@Component
public class ReportService {

    public ReportService() {
        System.out.println("ReportService created");
    }
}
```

Now Spring can postpone creation until the bean is needed.

```
ApplicationContext starts
       │
       ▼
BeanDefinition exists
       │
       ▼
No object created yet
       │
       │
       ▼
context.getBean(ReportService.class)
       │
       ▼
Create ReportService
       │
       ▼
Initialize
       │
       ▼
Return bean
```

### Important

Lazy initialization does **not** mean Spring doesn't know about the bean.

Spring still has the bean's definition/configuration metadata.

It postpones actual bean creation.

---

# 31\. Prototype Scope

Example:

```
@Component
@Scope("prototype")
public class ReportGenerator {
}
```

Every request for the bean can result in a new instance.

```
ReportGenerator r1 =
        context.getBean(ReportGenerator.class);

ReportGenerator r2 =
        context.getBean(ReportGenerator.class);
```

Conceptually:

```
getBean()
   │
   ▼
ReportGenerator #1

getBean()
   │
   ▼
ReportGenerator #2
```

Therefore:

```
r1 == r2
```

is normally:

```
false
```

---

# 32\. Singleton vs Prototype

```
                 SCOPE COMPARISON

┌────────────────────┬─────────────────────────────┐
│ Singleton          │ Prototype                   │
├────────────────────┼─────────────────────────────┤
│ Default scope      │ Explicitly configured       │
│ One per container  │ New instance per request    │
│ Usually eager      │ Created when requested      │
│ Container manages  │ Container creates/configures │
│ full lifecycle     │ but does not fully manage   │
│                    │ destruction lifecycle       │
└────────────────────┴─────────────────────────────┘
```

---

# 33\. Prototype Destruction — Very Important

Spring does **not normally invoke destruction callbacks for prototype beans**.

For example:

```
@Scope("prototype")
@Component
public class ReportGenerator {

    @PostConstruct
    public void init() {
        System.out.println("init");
    }

    @PreDestroy
    public void destroy() {
        System.out.println("destroy");
    }
}
```

The initialization callback can run.

But when the application context closes, Spring does not normally track every prototype instance and invoke:

```
@PreDestroy
```

on all of them.

### Why?

The lifecycle responsibility changes after Spring hands the prototype instance to the caller.

Conceptually:

```
Prototype requested
       ↓
Spring creates object
       ↓
Inject dependencies
       ↓
Initialize object
       ↓
Give object to caller
       ↓
Spring stops managing destruction
```

The statement:

> "Spring releases the reference immediately and therefore GC happens"

is too simplistic.

The more accurate statement is:

> Spring does not retain prototype instances for destruction callbacks as it does for managed singletons. The object becomes eligible for garbage collection when no reachable references remain.

If the application still holds a reference, GC cannot collect it.

---

# 34\. Destruction Lifecycle

When the application context shuts down:

```
ConfigurableApplicationContext context =
        new AnnotationConfigApplicationContext(AppConfig.class);

context.close();
```

Spring starts the destruction phase.

Conceptually:

```
ApplicationContext.close()
          │
          ▼
Singleton destruction
          │
          ▼
Destruction callbacks
          │
          ▼
Bean removed from container
```

---

# 35\. Destruction Callback #1 — @PreDestroy

Example:

```
@Component
public class CartService {

    @PreDestroy
    public void cleanup() {

        System.out.println(
            "Cleaning resources..."
        );
    }
}
```

When the context shuts down:

```
context.close()
      ↓
@PreDestroy
      ↓
cleanup()
```

---

# 36\. Destruction Callback #2 — DisposableBean

Example:

```
@Component
public class PaymentService
        implements DisposableBean {

    @Override
    public void destroy() {

        System.out.println(
            "Closing PaymentService"
        );
    }
}
```

Spring calls:

```
destroy();
```

---

# 37\. Destruction Callback #3 — destroyMethod

Example:

```
public class PaymentClient {

    public void stop() {
        System.out.println(
            "PaymentClient stopped"
        );
    }
}
```

Configuration:

```
@Bean(destroyMethod = "stop")
public PaymentClient paymentClient() {
    return new PaymentClient();
}
```

During context shutdown:

```
paymentClient.stop();
```

---

# 38\. Destruction Methods — Comparison

```
                    DESTRUCTION
                         │
         ┌───────────────┼────────────────┐
         │               │                │
         ▼               ▼                ▼
    @PreDestroy    DisposableBean    destroyMethod
         │               │                │
         ▼               ▼                ▼
    Annotation       Interface        @Bean config
```

Typical application code often favors:

```
@PreDestroy
```

when an annotation-based lifecycle callback is appropriate.

---

# 39\. Complete Example

Let's build one bean that demonstrates the major lifecycle stages.

```
@Component
public class OrderService
        implements BeanNameAware,
                   ApplicationContextAware,
                   InitializingBean,
                   DisposableBean {

    private PaymentService paymentService;

    public OrderService() {
        System.out.println("1. Constructor");
    }

    @Autowired
    public void setPaymentService(
            PaymentService paymentService) {

        this.paymentService = paymentService;

        System.out.println(
            "2. Dependency Injection"
        );
    }

    @Override
    public void setBeanName(String name) {

        System.out.println(
            "3. BeanNameAware: " + name
        );
    }

    @Override
    public void setApplicationContext(
            ApplicationContext context) {

        System.out.println(
            "4. ApplicationContextAware"
        );
    }

    @PostConstruct
    public void postConstruct() {

        System.out.println(
            "5. @PostConstruct"
        );
    }

    @Override
    public void afterPropertiesSet() {

        System.out.println(
            "6. afterPropertiesSet()"
        );
    }

    @PreDestroy
    public void preDestroy() {

        System.out.println(
            "7. @PreDestroy"
        );
    }

    @Override
    public void destroy() {

        System.out.println(
            "8. DisposableBean.destroy()"
        );
    }
}
```

---

# 40\. Lifecycle Output

The exact output can vary depending on the complete application and infrastructure, but conceptually:

```
Constructor
    ↓
Dependency Injection
    ↓
BeanNameAware
    ↓
ApplicationContextAware
    ↓
BeanPostProcessor.beforeInitialization()
    ↓
@PostConstruct
    ↓
afterPropertiesSet()
    ↓
custom initMethod
    ↓
BeanPostProcessor.afterInitialization()
    ↓
Bean Ready
    ↓
...
ApplicationContext.close()
    ↓
@PreDestroy
    ↓
DisposableBean.destroy()
    ↓
custom destroyMethod
```

Do not memorize only the output.

Understand **why each stage exists**.

---

# 41\. BeanPostProcessor vs Bean Lifecycle Callback

This is frequently asked in interviews.

## Bean lifecycle callback

Example:

```
@PostConstruct
public void init() {
}
```

This is generally about:

> "Initialize this particular bean."

## BeanPostProcessor

Example:

```
postProcessBeforeInitialization(...)
```

This is about:

> "Apply processing to beans during their lifecycle."

Conceptually:

```
                    Spring Container
                           │
             ┌─────────────┴─────────────┐
             │                           │
             ▼                           ▼
       Individual Bean            BeanPostProcessor
             │                           │
             │                    Processes beans
             ▼                           │
       @PostConstruct ◄──────────────────┘
```

A `BeanPostProcessor` can process many beans.

---

# 42\. `@PostConstruct` vs Constructor

Consider:

```
@Component
public class OrderService {

    @Autowired
    private PaymentService paymentService;

    public OrderService() {
        // paymentService is not available
    }

    @PostConstruct
    public void init() {
        // paymentService is available
    }
}
```

Lifecycle:

```
new OrderService()
      │
      ▼
paymentService = null
      │
      ▼
Dependency Injection
      │
      ▼
paymentService assigned
      │
      ▼
@PostConstruct
      │
      ▼
paymentService can be used
```

For constructor injection:

```
@Component
public class OrderService {

    private final PaymentService paymentService;

    public OrderService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }
}
```

The required dependency is available during construction.

---

# 43\. Circular Dependency

Suppose:

```
@Component
public class ClassA {

    private final ClassB b;

    public ClassA(ClassB b) {
        this.b = b;
    }
}
```

and:

```
@Component
public class ClassB {

    private final ClassA a;

    public ClassB(ClassA a) {
        this.a = a;
    }
}
```

Spring needs:

```
Create A
  ↓
A requires B
  ↓
Create B
  ↓
B requires A
  ↓
Create A
  ↓
A requires B
  ↓
...
```

This forms a cycle.

```
       ┌──────────────┐
       │              │
       ▼              │
    Class A ───────► Class B
       ▲              │
       └──────────────┘
```

Constructor injection makes the dependency cycle impossible to satisfy in the straightforward creation sequence.

---

# 44\. Circular Dependency with Setter/Field Injection

Historically, some circular dependencies involving setter/field injection can be resolved by Spring's singleton creation machinery.

For example:

```
@Component
public class ClassA {

    @Autowired
    private ClassB b;
}
```

```
@Component
public class ClassB {

    @Autowired
    private ClassA a;
}
```

However, whether a circular dependency can be resolved depends on the exact dependency type and Spring configuration.

### Important Interview Advice

Do not say:

> "Spring always supports circular dependencies with field injection."

That is incorrect.

A better answer:

> "Some circular dependencies involving singleton setter/field injection may be resolved through Spring's early singleton exposure mechanism, whereas constructor-based circular dependencies cannot be resolved in the same way."

---

# 45\. Is @PostConstruct a Good Circular Dependency Solution?

You may see examples such as:

```
@Component
public class ClassA {

    @Autowired
    private ClassB b;

    @PostConstruct
    public void init() {
        b.setA(this);
    }
}
```

This can sometimes be used to restructure when the reference is established.

But it should **not** be treated as the recommended solution.

Better approaches:

```
Circular dependency
       │
       ▼
Ask: Why does A need B?
Ask: Why does B need A?
       │
       ▼
Refactor responsibilities
       │
       ▼
Introduce another service/interface if appropriate
```

### Interview Answer

> "Using `@PostConstruct` or setter injection can sometimes avoid a constructor cycle, but eliminating the circular dependency through better design is preferable."

---

# 46\. Eager Singleton vs Lazy Singleton vs Prototype

| Feature | Eager Singleton | Lazy Singleton | Prototype |
| --- | --- | --- | --- |
| Default? | Yes | No | No |
| Instance count | One per container | One per container | New instance per request |
| Creation | Startup | First required use | Each request |
| Dependencies injected | Yes | Yes | Yes |
| Initialization callbacks | Yes | Yes | Yes |
| Destruction callback | Yes, when managed context closes | Yes | Normally no |
| `@Lazy` | No | Yes | Not the main mechanism |
| Typical use | Shared application service | Expensive rarely used service | Short-lived stateful object |

---

# 47\. Lifecycle Diagram by Scope

```
                     SPRING SCOPES
                         │
          ┌──────────────┼───────────────┐
          │              │               │
          ▼              ▼               ▼
      Singleton         Lazy          Prototype
          │              │               │
          ▼              ▼               ▼
   Create at startup   Create when     Create every
   by default          first needed    time requested
          │              │               │
          ▼              ▼               ▼
       Inject          Inject          Inject
          │              │               │
          ▼              ▼               ▼
       Initialize      Initialize      Initialize
          │              │               │
          ▼              ▼               ▼
        Use             Use             Use
          │              │               │
          ▼              ▼               ▼
    Context close    Context close     Caller manages
          │              │             eventual cleanup
          ▼              ▼
       Destroy        Destroy
```

---

# 48\. Important: Bean Lifecycle ≠ JVM Object Lifecycle

This is a common conceptual mistake.

Spring lifecycle:

```
Bean creation
      ↓
Bean initialization
      ↓
Bean usage
      ↓
Spring destruction callback
```

JVM lifecycle:

```
Object becomes unreachable
      ↓
Eligible for GC
      ↓
Garbage Collector may reclaim memory
```

These are different concepts.

```
Spring
  │
  └── manages bean lifecycle

JVM
  │
  └── manages memory / garbage collection
```

Spring does not perform garbage collection.

---

# 49\. Who Owns What?

A useful mental model:

```
Spring Container
│
├── Creates beans
├── Injects dependencies
├── Initializes beans
├── Stores managed singleton beans
└── Calls destruction callbacks
        │
        ▼
JVM
│
└── Garbage Collector manages memory reclamation
```

---

# 50\. Why Does Spring Need Lifecycle Callbacks?

Imagine a service that needs to:

- Load configuration
- Build an in-memory cache
- Validate dependencies
- Open a resource
- Start a client
- Register itself somewhere

These tasks may require dependencies to already exist.

Therefore:

```
Constructor
    ↓
Dependencies available
    ↓
@PostConstruct
    ↓
Initialize dependent resources
```

Similarly, resources may need cleanup:

```
Application running
      ↓
Context closing
      ↓
@PreDestroy
      ↓
Close resources
```

---

# 51\. Real-World Example — Cache Initialization

```
@Component
public class ProductCache {

    private final ProductRepository repository;

    private Map<Long, String> cache;

    public ProductCache(ProductRepository repository) {
        this.repository = repository;
    }

    @PostConstruct
    public void initialize() {

        cache = new HashMap<>();

        repository.findAll()
                .forEach(product ->
                    cache.put(
                        product.getId(),
                        product.getName()
                    )
                );
    }

    public String getProductName(Long id) {
        return cache.get(id);
    }
}
```

Lifecycle:

```
ProductRepository created
          ↓
ProductCache created
          ↓
Repository injected
          ↓
@PostConstruct
          ↓
Load product data
          ↓
Cache ready
```

---

# 52\. Real-World Example — Resource Cleanup

```
@Component
public class ExternalClient {

    private Connection connection;

    @PostConstruct
    public void connect() {

        connection = createConnection();

        System.out.println("Connected");
    }

    @PreDestroy
    public void disconnect() {

        if (connection != null) {
            connection.close();
        }

        System.out.println("Disconnected");
    }

    private Connection createConnection() {
        // create connection
        return null;
    }
}
```

Lifecycle:

```
Application Startup
        │
        ▼
@PostConstruct
        │
        ▼
Open connection
        │
        ▼
Application runs
        │
        ▼
ApplicationContext.close()
        │
        ▼
@PreDestroy
        │
        ▼
Close connection
```

---

# 53\. `@Bean(initMethod, destroyMethod)`

A useful example:

```
@Configuration
public class AppConfig {

    @Bean(
        initMethod = "start",
        destroyMethod = "stop"
    )
    public ExternalClient externalClient() {
        return new ExternalClient();
    }
}
```

Class:

```
public class ExternalClient {

    public void start() {
        System.out.println("Client started");
    }

    public void stop() {
        System.out.println("Client stopped");
    }
}
```

Lifecycle:

```
new ExternalClient()
        ↓
Dependency processing
        ↓
start()
        ↓
Bean ready
        ↓
context.close()
        ↓
stop()
```

This is especially useful when working with classes you cannot annotate or modify.

---

# 54\. A Practical Lifecycle Cheat Sheet

```
┌──────────────────────────────────────────────────────┐
│                SPRING BEAN LIFECYCLE                 │
├──────────────────────────────────────────────────────┤
│                                                      │
│  1. Configuration loaded                             │
│  2. BeanDefinition registered                        │
│  3. Bean instantiated                               │
│  4. Dependencies injected                            │
│  5. Aware callbacks                                  │
│  6. postProcessBeforeInitialization()               │
│  7. @PostConstruct                                   │
│  8. afterPropertiesSet()                             │
│  9. custom init-method                               │
│ 10. postProcessAfterInitialization()                │
│ 11. Bean ready                                       │
│                                                      │
│             APPLICATION RUNNING                      │
│                                                      │
│ 12. ApplicationContext.close()                       │
│ 13. @PreDestroy                                      │
│ 14. DisposableBean.destroy()                         │
│ 15. custom destroy-method                            │
│ 16. Bean no longer managed by container              │
│                                                      │
└──────────────────────────────────────────────────────┘
```

---

# 55\. Interview Questions

## Q1. What is the Spring Bean Lifecycle?

**Answer:**

The Spring Bean Lifecycle describes the process through which Spring:

```
Creates
  ↓
Injects dependencies
  ↓
Initializes
  ↓
Uses/manages
  ↓
Destroys
```

a bean.

Typical stages include instantiation, dependency injection, Aware callbacks, BeanPostProcessor callbacks, initialization callbacks such as `@PostConstruct`, and destruction callbacks such as `@PreDestroy`.

---

## Q2. What is a BeanDefinition?

**Answer:**

A `BeanDefinition` is metadata describing how Spring should create and configure a bean.

It can contain information such as:

- Bean class
- Bean name
- Scope
- Lazy configuration
- Constructor information
- Dependencies
- Initialization/destruction configuration

Think:

```
BeanDefinition = Blueprint
Bean = Actual Object
```

---

## Q3. What is the default scope of a Spring bean?

**Answer:**

`singleton`.

There is normally one managed instance of the bean per Spring IoC container.

---

## Q4. What is the difference between singleton and prototype?

**Answer:**

Singleton:

```
One instance per container
```

Prototype:

```
A new instance is created for each request to the container.
```

Prototype beans receive initialization processing, but Spring normally does not manage their destruction callbacks.

---

## Q5. What is `@PostConstruct`?

**Answer:**

`@PostConstruct` marks a method to be invoked during bean initialization after dependency injection has occurred.

Example:

```
@PostConstruct
public void init() {
    // initialization logic
}
```

---

## Q6. What is `@PreDestroy`?

**Answer:**

`@PreDestroy` marks a method that Spring invokes during destruction of a managed bean, typically when can sometimes be used in designs involving setter/field injection to alter when a reference injection cannot use the same mechanism because the constructor itself requires the dependency startup work may be an explicit startup strategy or lazy/asynchronous approach depending on and registers BeanDefinitions. For a bean that needs to be created, Spring instantiates it and injects it manages. Prototype beans are different because Spring normally does not manage their the application context shuts down.

Example:

```
@PreDestroy
public void cleanup() {
    // cleanup logic
}
```

---

## Q7. Does `@PreDestroy` execute for prototype beans?

**Answer:**

Normally, no.

Spring creates and initializes prototype instances but does not normally retain them for destruction callbacks.

Therefore:

```
Singleton → Spring manages destruction
Prototype → Caller is responsible for eventual cleanup
```

---

## Q8. Why is `@PostConstruct` useful?

**Answer:**

It is useful when initialization requires dependencies that must already have been injected.

Example:

```
@Autowired
private PaymentService paymentService;

@PostConstruct
public void init() {
    paymentService.initialize();
}
```

---

## Q9. Can we use dependencies inside a constructor?

**Answer:**

Yes, with constructor injection.

Example:

```
public OrderService(PaymentService paymentService) {
    this.paymentService = paymentService;
}
```

This is one reason constructor injection is preferred.

However, with field/setter injection, those dependencies have not yet been injected when the constructor runs.

---

## Q10. What is `InitializingBean`?

**Answer:**

`InitializingBean` is a Spring lifecycle interface containing:

```
afterPropertiesSet()
```

Example:

```
@Component
public class MyService
        implements InitializingBean {

    @Override
    public void afterPropertiesSet() {
        System.out.println("Initialized");
    }
}
```

---

## Q11. What is `DisposableBean`?

**Answer:**

It is a Spring lifecycle interface containing:

```
destroy()
```

Example:

```
@Component
public class MyService
        implements DisposableBean {

    @Override
    public void destroy() {
        System.out.println("Destroyed");
    }
}
```

---

## Q12. What is the difference between `@PostConstruct` and `InitializingBean`?

**Answer:**

Both are initialization callbacks.

```
@PostConstruct
    ↓
Annotation-based

InitializingBean
    ↓
Spring interface-based
```

`@PostConstruct` generally avoids coupling the class directly to a Spring lifecycle interface.

---

## Q13. What is `initMethod`?

**Answer:**

It specifies a custom initialization method on a `@Bean`.

Example:

```
@Bean(initMethod = "start")
public PaymentClient paymentClient() {
    return new PaymentClient();
}
```

Spring invokes:

```
start();
```

---

## Q14. What is `destroyMethod`?

**Answer:**

It specifies a custom destruction method for a `@Bean`.

```
@Bean(destroyMethod = "stop")
public PaymentClient paymentClient() {
    return new PaymentClient();
}
```

When the context closes, Spring invokes:

```
stop();
```

---

## Q15. What is BeanPostProcessor?

**Answer:**

`BeanPostProcessor` is an extension point that allows Spring to process bean instances before and after initialization.

Important methods:

```
postProcessBeforeInitialization()
```

and:

```
postProcessAfterInitialization()
```

It is important in understanding many Spring infrastructure features and proxy creation.

---

## Q16. Does BeanPostProcessor run before or after `@PostConstruct`?

**Answer:**

For the standard initialization flow:

```
postProcessBeforeInitialization()
        ↓
@PostConstruct
        ↓
afterPropertiesSet()
        ↓
custom init-method
        ↓
postProcessAfterInitialization()
```

This is an important interview sequence to remember.

---

## Q17. What are Aware interfaces?

**Answer:**

Aware interfaces allow a bean to receive certain container-related information.

Examples:

```
BeanNameAware
ApplicationContextAware
BeanFactoryAware
EnvironmentAware
```

For example:

```
BeanNameAware
    ↓
setBeanName(...)
```

---

## Q18. What is `ApplicationContextAware`?

**Answer:**

It allows a bean to receive the `ApplicationContext`.

Example:

```
@Component
public class MyService
        implements ApplicationContextAware {

    @Override
    public void setApplicationContext(
            ApplicationContext context) {
    }
}
```

Use it carefully because directly accessing the container can increase coupling.

---

## Q19. What happens when `ApplicationContext.close()` is called?

**Answer:**

For a closable application context, Spring begins shutdown processing and invokes destruction callbacks for beans it manages, particularly singleton beans.

Conceptually:

```
context.close()
      ↓
Destruction callbacks
      ↓
Singleton beans destroyed
      ↓
Context shutdown
```

---

## Q20. How do you trigger bean destruction manually?

Usually, close the application context:

```
ConfigurableApplicationContext context =
        new AnnotationConfigApplicationContext(AppConfig.class);

context.close();
```

---

# 56\. Tricky Interview Questions

## Q21. Is Spring singleton the same as Singleton Design Pattern?

**Answer:**

No.

Spring singleton means:

> One managed instance per Spring container.

The Singleton Design Pattern is a class-level design pattern that restricts object creation.

Spring manages the instance through the IoC container.

---

## Q22. Does Spring perform Garbage Collection?

**Answer:**

No.

The JVM Garbage Collector manages memory reclamation.

Spring manages bean lifecycle and invokes lifecycle callbacks.

```
Spring → Bean lifecycle
JVM    → Memory management / GC
```

---

## Q23. Does destroying a bean mean Java immediately frees its memory?

**Answer:**

No.

After Spring stops managing the bean and there are no reachable references, the object can become eligible for garbage collection.

GC timing is controlled by the JVM.

---

## Q24. Does `@PostConstruct` run before dependency injection?

**Answer:**

No.

For normal bean creation:

```
Instantiation
    ↓
Dependency Injection
    ↓
@PostConstruct
```

Therefore injected dependencies can normally be used from the `@PostConstruct` method.

---

## Q25. Does the constructor run after dependency injection?

**Answer:**

No.

The constructor is involved in creating the object.

For field/setter injection:

```
Constructor
    ↓
Dependency Injection
```

For constructor injection, dependencies are supplied as constructor arguments.

---

## Q26. Can a prototype bean have `@PostConstruct`?

**Answer:**

Yes.

Prototype beans still go through creation, dependency injection, and initialization processing.

The important difference is destruction management.

```
Prototype
   ↓
Create
   ↓
Inject
   ↓
@PostConstruct
   ↓
Return to caller
   ↓
No normal Spring destruction callback management
```

---

## Q27. Why is constructor injection generally preferred?

**Answer:**

Because it:

- Makes dependencies explicit
- Supports immutable fields
- Makes required dependencies mandatory
- Improves testability
- Reduces partially initialized objects
- Helps expose design problems such as circular dependencies

---

## Q28. Can `@PostConstruct` solve a circular dependency?

**Answer:**

It can sometimes be used in designs involving setter/field injection to alter when a reference is established, but it is not the preferred solution.

The better solution is generally to redesign the dependency graph.

---

## Q29. Why can setter/field circular dependencies sometimes work while constructor circular dependencies fail?

**Answer:**

Spring's singleton creation machinery can sometimes expose an early reference to a singleton before its complete initialization.

Constructor injection cannot use the same mechanism because the constructor itself requires the dependency before the object can be fully instantiated.

Conceptually:

```
Constructor cycle:

A needs B
B needs A
↓
Cannot finish constructing A
```

Whereas some setter/field cases can conceptually proceed:

```
Create A
  ↓
Early A reference
  ↓
Create B
  ↓
Inject A into B
  ↓
Inject B into A
```

The exact behavior depends on the dependency and Spring configuration.

---

# 57\. Scenario-Based Interview Questions

## Scenario 1

You have:

```
@Component
public class A {

    @Autowired
    private B b;

    @PostConstruct
    public void init() {
        System.out.println(b);
    }
}
```

### Question

Will `b` normally be null inside `init()`?

### Answer

No.

For standard field injection, dependency injection occurs before the initialization callback.

So:

```
Constructor
   ↓
Inject B
   ↓
@PostConstruct
```

---

# 58\. Scenario 2

```
@Component
public class A {

    @Autowired
    private B b;

    public A() {
        System.out.println(b);
    }
}
```

### Question

Will `b` be available?

### Answer

No, not with field injection.

The constructor executes before the field is injected.

---

# 59\. Scenario 3

```
@Component
@Scope("prototype")
public class A {

    @PreDestroy
    public void cleanup() {
        System.out.println("cleanup");
    }
}
```

Then:

```
context.getBean(A.class);
context.close();
```

### Question

Will `cleanup()` normally execute?

### Answer

No.

Spring normally does not manage destruction callbacks for prototype instances.

---

# 60\. Scenario 4

```
@Component
@Lazy
public class A {

    public A() {
        System.out.println("Created");
    }
}
```

### Question

When is the constructor called?

### Answer

Creation is postponed until the bean is actually needed, such as through a `getBean()` request or another dependency requiring the bean.

---

# 61\. Scenario 5

Suppose:

```
@Component
public class A implements BeanNameAware {

    @Override
    public void setBeanName(String name) {
        System.out.println(name);
    }
}
```

### Question

What does Spring provide?

### Answer

The bean's registered name.

For example:

```
a
```

depending on the bean naming rules and configuration.

---

# 62\. Scenario 6 — Identify the Order

Given:

```
A. @PostConstruct
B. Constructor
C. postProcessAfterInitialization()
D. Dependency Injection
E. postProcessBeforeInitialization()
```

Correct simplified order:

```
B → D → E → A → C
```

---

# 63\. Scenario 7 — Initialization Interfaces

Suppose a bean has all three:

```
@PostConstruct
public void postConstruct() {}
```

```
@Override
public void afterPropertiesSet() {}
```

and:

```
@Bean(initMethod = "start")
```

### Question

What is the conceptual order?

```
@PostConstruct
       ↓
afterPropertiesSet()
       ↓
custom init-method
```

with `BeanPostProcessor` stages surrounding initialization as described earlier.

---

# 64\. Scenario 8 — Why Not Put Everything in Constructor?

Bad approach:

```
public OrderService() {

    // huge file loading
    // database calls
    // network calls
    // expensive cache building
}
```

Better:

```
public OrderService(
        OrderRepository repository) {

    this.repository = repository;
}

@PostConstruct
public void init() {

    // initialization requiring
    // injected dependencies
}
```

Even better for truly expensive startup work may be an explicit startup strategy or lazy/asynchronous approach depending on application requirements.

---

# 65\. Common Mistakes to Avoid

## Mistake 1

> "Spring uses reflection for everything."

Better:

> Spring uses a sophisticated bean creation infrastructure with reflection and multiple instantiation strategies.

---

## Mistake 2

> "Prototype objects are immediately garbage collected."

Wrong.

Correct:

> Spring doesn't normally manage prototype destruction. The object becomes eligible for GC only when no reachable references remain.

---

## Mistake 3

> "Spring singleton means one object in the entire JVM."

Wrong.

Correct:

> One managed instance per Spring container.

---

## Mistake 4

> "`@PostConstruct` executes before dependency injection."

Wrong.

Correct:

```
Dependency Injection
        ↓
@PostConstruct
```

---

## Mistake 5

> "Circular dependencies always work with field injection."

Wrong.

Some circular dependencies may be resolvable, but not all.

---

## Mistake 6

> "`@PostConstruct` is always the place for heavy operations."

Not necessarily.

Heavy startup operations can slow application startup.

---

# 66\. Best Practices

### Prefer Constructor Injection

```
@Component
public class OrderService {

    private final PaymentService paymentService;

    public OrderService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }
}
```

### Keep Constructors Simple

Use constructors to establish object invariants and assign required dependencies.

### Use `@PostConstruct` for Post-Injection Initialization

```
@PostConstruct
public void init() {
    // initialization requiring dependencies
}
```

### Use `@PreDestroy` for Cleanup

```
@PreDestroy
public void cleanup() {
    // cleanup
}
```

### Avoid Circular Dependencies

Instead of:

```
A → B
↑   ↓
└───┘
```

prefer:

```
A → C ← B
```

where `C` owns shared responsibility when appropriate.

---

# 67\. One-Page Interview Revision

```
                    SPRING BEAN LIFECYCLE
                           │
                           ▼
                  Configuration Loaded
                           │
                           ▼
                    BeanDefinition
                           │
                           ▼
                     Instantiation
                           │
                           ▼
                 Dependency Injection
                           │
                           ▼
                    Aware Interfaces
                           │
                           ▼
          BeanPostProcessor.beforeInitialization()
                           │
                           ▼
                     @PostConstruct
                           │
                           ▼
                afterPropertiesSet()
                           │
                           ▼
                     initMethod
                           │
                           ▼
          BeanPostProcessor.afterInitialization()
                           │
                           ▼
                      Bean Ready
                           │
                           ▼
                    Application Runs
                           │
                           ▼
               ApplicationContext.close()
                           │
                           ▼
                     @PreDestroy
                           │
                           ▼
                       destroy()
                           │
                           ▼
                    destroyMethod
                           │
                           ▼
                  Spring stops managing
                           │
                           ▼
                JVM may eventually GC
```

---

# 68\. Ultra-Short Memory Trick

Remember:

```
C → D → A → B → I → U → D
```

Where:

```
C = Constructor
D = Dependency Injection
A = Aware
B = BeanPostProcessor
I = Initialization
U = Use
D = Destruction
```

More accurately for interviews:

```
Constructor
    ↓
Dependency Injection
    ↓
Aware
    ↓
BeforeInitialization
    ↓
@PostConstruct
    ↓
afterPropertiesSet
    ↓
initMethod
    ↓
AfterInitialization
    ↓
Bean Ready
    ↓
@PreDestroy
    ↓
destroy
    ↓
destroyMethod
```

---

# 69\. Final Interview Answer — 60 Seconds

If an interviewer asks:

> **"Explain the Spring Bean Lifecycle."**

A strong answer is:

> "The Spring Bean Lifecycle starts when the IoC container processes configuration and registers BeanDefinitions. For a bean that needs to be created, Spring instantiates it and injects its dependencies. It then invokes relevant Aware callbacks, followed by `BeanPostProcessor` processing before initialization. Initialization callbacks such as `@PostConstruct`, `InitializingBean.afterPropertiesSet()`, and a configured init method can then execute. After `BeanPostProcessor`'s after-initialization phase, the bean is ready for use. When the application context is closed, Spring invokes destruction callbacks such as `@PreDestroy`, `DisposableBean.destroy()`, and configured destroy methods for beans whose destruction lifecycle it manages. Prototype beans are different because Spring normally does not manage their destruction callbacks."

---

# 70\. Final Mental Model

The easiest way to understand everything is:

```
                 SPRING CONTAINER
                        │
                        ▼
               "What beans exist?"
                        │
                        ▼
                 BeanDefinitions
                        │
                        ▼
               "Create this bean"
                        │
                        ▼
                  Constructor
                        │
                        ▼
               "Give it dependencies"
                        │
                        ▼
               Dependency Injection
                        │
                        ▼
              "Give it container info"
                        │
                        ▼
                 Aware Callbacks
                        │
                        ▼
             "Process the bean"
                        │
                        ▼
             BeanPostProcessor
                        │
                        ▼
              "Initialize the bean"
                        │
                        ▼
        @PostConstruct / afterPropertiesSet
                        │
                        ▼
                Bean Ready
                        │
                        ▼
                  APPLICATION
                        │
                        ▼
             "Application shutting down"
                        │
                        ▼
                @PreDestroy
                        │
                        ▼
                 destroy()
                        │
                        ▼
              Bean no longer managed
                        │
                        ▼
             JVM eventually handles GC
```

## ⭐ Most Important Things to Remember for Interviews

1. **BeanDefinition is metadata, not the actual bean.**
2. **Spring singleton = one instance per container.**
3. **Constructor runs before field/setter injection.**
4. **`@PostConstruct` runs after dependency injection.**
5. **`BeanPostProcessor` has before- and after-initialization hooks.**
6. **`@PreDestroy` is normally relevant to beans whose destruction Spring manages.**
7. **Prototype beans are initialized by Spring but normally aren't destroyed by Spring.**
8. **Spring does not perform Garbage Collection.**
9. **Constructor injection is generally preferred.**
10. **Circular dependencies should preferably be removed through refactoring rather than worked around.**
11. **`@PostConstruct`, `InitializingBean`, and `initMethod` are initialization mechanisms.**
12. **`@PreDestroy`, `DisposableBean`, and `destroyMethod` are destruction mechanisms.**
13. **`ApplicationContext.close()` starts the context shutdown/destruction process.**
14. **Lazy singleton delays actual bean creation, not registration of its definition.**
15. **Understand the lifecycle conceptually rather than memorizing console output.**

---

## Quick Revision Table

| Topic | Remember |
| --- | --- |
| IoC | Spring controls object creation |
| BeanDefinition | Bean blueprint/metadata |
| Singleton | One instance per container |
| Prototype | New instance per request |
| Constructor | Object creation |
| DI | Dependencies are supplied |
| Aware | Container/environment information |
| BPP | Processes beans before/after initialization |
| `@PostConstruct` | Post-injection initialization |
| `InitializingBean` | `afterPropertiesSet()` |
| `initMethod` | Custom initialization method |
| `@PreDestroy` | Cleanup callback |
| `DisposableBean` | `destroy()` |
| `destroyMethod` | Custom destruction method |
| Lazy | Delays bean creation |
| GC | JVM responsibility |
| Circular dependency | Prefer refactoring |
