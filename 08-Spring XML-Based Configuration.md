# Spring XML-Based Configuration

---

## 1\. Introduction

Spring provides multiple ways to provide **configuration metadata** to the IoC container.

The two commonly encountered approaches are:

1. **Annotation-based configuration**
2. **XML-based configuration**

Modern Spring Boot applications primarily use annotations and Java configuration, but XML configuration is still important when:

- Maintaining legacy Spring applications
- Working with enterprise applications built before annotation-based configuration became standard
- Migrating an old application to Spring Boot
- Working with hybrid XML + annotation applications
- Understanding the evolution of the Spring Framework
- Configuring classes without modifying their Java source code

### Core idea

Regardless of whether configuration comes from XML, annotations, or Java configuration, Spring ultimately needs **bean metadata**.

```
                         SPRING IoC CONTAINER
                                  │
                                  │
                         Configuration Metadata
                                  │
              ┌───────────────────┼───────────────────┐
              │                   │                   │
              ▼                   ▼                   ▼
        XML Configuration    Java Configuration   Annotation Config
        <bean>               @Bean               @Component
        <property>           @Configuration      @Service
        <constructor-arg>                         @Repository
              │                   │                   │
              └───────────────────┼───────────────────┘
                                  ▼
                         BeanDefinition Objects
                                  │
                                  ▼
                         Bean Instantiation
                                  │
                                  ▼
                       Dependency Injection
                                  │
                                  ▼
                         Lifecycle Management
```

The **configuration format changes**, but the fundamental IoC process remains the same.

---

# 2\. XML vs Annotation Configuration

## Annotation-Based Configuration

Example:

```
@Component
public class OrderService {
}
```

Or:

```
@Configuration
public class AppConfig {

    @Bean
    public OrderService orderService() {
        return new OrderService();
    }
}
```

Container:

```
ApplicationContext context =
        new AnnotationConfigApplicationContext(AppConfig.class);
```

---

## XML-Based Configuration

XML:

```
<bean id="orderService"
      class="in.strikes.OrderService"/>
```

Container:

```
ApplicationContext context =
        new ClassPathXmlApplicationContext("beans.xml");
```

---

## Comparison

| Feature | Annotation | XML |
| --- | --- | --- |
| Bean declaration | `@Component`, `@Bean` | `<bean>` |
| Dependency injection | `@Autowired`, constructor | `<constructor-arg>`, `<property>` |
| Component scanning | `@ComponentScan` | `<context:component-scan>` |
| Lifecycle | `@PostConstruct`, etc. | `init-method`, `destroy-method` |
| Scope | `@Scope` | `scope` attribute |
| Configuration | Java source | XML file |
| Modifying Java classes | Usually required | Often not required |
| Legacy support | Less common | Very common |
| Modern Spring Boot | Preferred | Usually legacy/specialized |

---

# 3\. Project Setup

A simple Spring Core XML project can be created using Maven.

Example structure:

```
xml-based-config-demo/
│
├── pom.xml
│
└── src/
    └── main/
        ├── java/
        │   └── in/
        │       └── strikes/
        │           ├── Main.java
        │           ├── OrderService.java
        │           └── PaymentService.java
        │
        └── resources/
            └── beans.xml
```

---

## Maven Dependency

For a Spring 6.x project:

```
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>6.x.x</version>
</dependency>
```

`spring-context` brings in the core Spring modules required for normal application-context usage.

> **Version note:** Always prefer a currently supported Spring version compatible with your JDK and project requirements rather than copying an old version blindly.

---

# 4\. What Is the Classpath?

When using:

```
new ClassPathXmlApplicationContext("beans.xml");
```

Spring looks for `beans.xml` on the application's **classpath**.

In a standard Maven project:

```
src/main/resources/
```

is copied to the classpath during the build.

Therefore:

```
src/main/resources/beans.xml
```

can be loaded using:

```
new ClassPathXmlApplicationContext("beans.xml");
```

### Example

```
src/main/resources/
└── beans.xml
```

Java:

```
ApplicationContext context =
        new ClassPathXmlApplicationContext("beans.xml");
```

---

## If the file is inside a directory

Suppose:

```
src/main/resources/
└── config/
    └── beans.xml
```

Then:

```
ApplicationContext context =
        new ClassPathXmlApplicationContext("config/beans.xml");
```

---

## Incorrect File Name

If the application expects:

```
new ClassPathXmlApplicationContext("beans.xml");
```

but the actual file is:

```
bean.xml
```

Spring cannot find the resource and context initialization fails.

---

# 5\. Creating the XML Configuration File

A traditional Spring XML configuration file looks like:

```
<?xml version="1.0" encoding="UTF-8"?>

<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="
           http://www.springframework.org/schema/beans
           http://www.springframework.org/schema/beans/spring-beans.xsd">

</beans>
```

The namespace and schema tell Spring how to interpret elements such as:

```
<bean>
<property>
<constructor-arg>
```

You generally don't need to memorize the entire header.

---

# 6\. Defining a Bean Using `<bean>`

Suppose we have:

```
package in.strikes;

public class OrderService {

    public void placeOrder() {
        System.out.println("Order placed");
    }
}
```

XML:

```
<bean id="orderService"
      class="in.strikes.OrderService"/>
```

This tells Spring:

> Create and manage an instance of `in.strikes.OrderService` and register it under the bean name `orderService`.

---

## Understanding `id`

```
<bean id="orderService"
      class="in.strikes.OrderService"/>
```

`id` is the primary bean identifier.

We can retrieve it using:

```
OrderService order =
        context.getBean("orderService", OrderService.class);
```

---

## Understanding `class`

```
class="in.strikes.OrderService"
```

This is the **fully qualified class name**.

It consists of:

```
package + class name
```

Example:

```
in.strikes.OrderService
```

---

# 7\. Java Configuration vs XML Configuration

## Java Configuration

```
@Bean
public OrderService orderService() {
    return new OrderService();
}
```

Equivalent XML:

```
<bean id="orderService"
      class="in.strikes.OrderService"/>
```

Conceptually:

```
Java Configuration                     XML Configuration

@Bean                                  <bean>
  │                                      │
  ▼                                      ▼
Bean Definition                        Bean Definition
  │                                      │
  └──────────────────┬───────────────────┘
                     ▼
                Spring IoC
```

---

# 8\. Default Bean Behavior

A normal XML bean:

```
<bean id="orderService"
      class="in.strikes.OrderService"/>
```

has:

```
Scope = singleton
Lazy initialization = false
```

Therefore, for a normal `ApplicationContext`, Spring generally creates the singleton bean during context initialization.

```
ApplicationContext starts
          │
          ▼
Read beans.xml
          │
          ▼
Create BeanDefinitions
          │
          ▼
Create singleton OrderService
          │
          ▼
Store singleton in container
```

---

# 9\. Retrieving Beans Using `getBean()`

There are three commonly used forms.

---

## Method 1 — By Bean Name

```
OrderService order =
        (OrderService) context.getBean("orderService");
```

Problem:

- Returns `Object`
- Requires casting

```
getBean("orderService")
          │
          ▼
       Object
          │
          ▼
     Type casting
          │
          ▼
    OrderService
```

If the bean does not exist:

```
NoSuchBeanDefinitionException
```

---

# 10\. Method 2 — By Type

```
OrderService order =
        context.getBean(OrderService.class);
```

Advantages:

- No explicit cast
- Type-safe

However, Spring needs to find a suitable bean of that type.

Suppose:

```
<bean id="orderService1"
      class="in.strikes.OrderService"/>

<bean id="orderService2"
      class="in.strikes.OrderService"/>
```

Then:

```
context.getBean(OrderService.class);
```

is ambiguous.

Spring may throw:

```
NoUniqueBeanDefinitionException
```

because there are multiple candidates.

---

# 11\. Method 3 — By Name + Type

Recommended when you know the bean name:

```
OrderService order =
        context.getBean("orderService", OrderService.class);
```

Benefits:

- No explicit cast
- Bean name identifies the exact bean
- Expected type is explicitly supplied

```
                 getBean()
                    │
          ┌─────────┴─────────┐
          │                   │
        Name                 Type
          │                   │
          └─────────┬─────────┘
                    ▼
             Exact Bean
                    │
                    ▼
             OrderService
```

---

# 12\. Important `getBean()` Exceptions

## `NoSuchBeanDefinitionException`

Usually means Spring cannot find a bean matching the requested name/type.

Example:

```
context.getBean("wrongName");
```

---

## `NoUniqueBeanDefinitionException`

Occurs when a type-based lookup has multiple matching candidates and Spring cannot determine which one should be returned.

Example:

```
context.getBean(OrderService.class);
```

when multiple `OrderService` beans exist.

---

# 13\. Bean ID and Bean Name

Spring XML supports both:

```
<bean id="orderService"
      name="order,orders,os"
      class="in.strikes.OrderService"/>
```

Here:

- `id` is the primary identifier
- `name` can provide aliases

Conceptually:

```
                OrderService Bean
                       │
              ┌────────┼────────┐
              │        │        │
              ▼        ▼        ▼
       orderService   order    orders
                         │
                         ▼
                        os
```

Aliases can be used to retrieve the same bean.

```
context.getBean("order");
context.getBean("orders");
context.getBean("os");
```

All refer to the same underlying singleton bean.

---

# 14\. Does Spring Generate a Default ID?

If you write:

```
<bean class="in.strikes.OrderService"/>
```

you have not explicitly assigned a bean name.

Do **not** assume that Spring simply creates the name:

```
orderService
```

and then rely on that name in your application.

For reliable configuration, explicitly provide:

```
<bean id="orderService"
      class="in.strikes.OrderService"/>
```

If you need to retrieve an unnamed bean, type-based lookup can work when the type is unambiguous.

---

# 15\. Dependency Injection in XML

Spring XML supports the major DI styles:

```
                 Dependency Injection
                         │
              ┌──────────┴──────────┐
              │                     │
              ▼                     ▼
       Constructor DI          Setter DI
       <constructor-arg>       <property>
```

XML does not directly perform traditional field injection such as:

```
@Autowired
private PaymentService paymentService;
```

because XML configuration normally expresses dependencies through constructors or JavaBean setter properties.

---

# 16\. Constructor Injection

Suppose:

```
public class OrderService {

    private PaymentService paymentService;

    public OrderService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }
}
```

And:

```
public class PaymentService {
}
```

XML:

```
<bean id="paymentService"
      class="in.strikes.PaymentService"/>

<bean id="orderService"
      class="in.strikes.OrderService">

    <constructor-arg ref="paymentService"/>

</bean>
```

---

## What happens internally?

```
beans.xml
   │
   ▼
Create PaymentService
   │
   ▼
Store PaymentService
   │
   ▼
Create OrderService
   │
   │ constructor requires PaymentService
   ▼
Pass PaymentService reference
   │
   ▼
OrderService created
```

The important point is that:

```
ref="paymentService"
```

means:

> Inject the Spring-managed bean named `paymentService`.

It does **not** mean:

> Create a new PaymentService manually.

---

# 17\. `ref` vs `value`

This distinction is extremely important.

## `ref`

Used when injecting another Spring bean.

```
<constructor-arg ref="paymentService"/>
```

Meaning:

```
Inject another Spring-managed object
```

---

## `value`

Used for literal configuration values.

```
<constructor-arg value="UPI"/>
<constructor-arg value="3"/>
```

Meaning:

```
String = "UPI"
int = 3
```

---

# 18\. Constructor Injection with Primitive/String Values

Java:

```
public class PaymentService {

    private String type;
    private int retryCount;

    public PaymentService(String type, int retryCount) {
        this.type = type;
        this.retryCount = retryCount;
    }
}
```

XML:

```
<bean id="paymentService"
      class="in.strikes.PaymentService">

    <constructor-arg value="UPI"/>
    <constructor-arg value="3"/>

</bean>
```

Spring performs the required type conversion.

Conceptually:

```
"UPI" ──────────────► String

"3" ── conversion ──► int
```

---

# 19\. Constructor Argument Index

Suppose:

```
public PaymentService(String type, int retryCount) {
}
```

You can explicitly specify positions:

```
<bean id="paymentService"
      class="in.strikes.PaymentService">

    <constructor-arg index="0"
                     value="UPI"/>

    <constructor-arg index="1"
                     value="3"/>

</bean>
```

Indexes are **zero-based**.

```
Constructor

index 0 ──► type
index 1 ──► retryCount
```

---

# 20\. Constructor Argument Name

You can also use:

```
<bean id="paymentService"
      class="in.strikes.PaymentService">

    <constructor-arg name="type"
                     value="UPI"/>

    <constructor-arg name="retryCount"
                     value="3"/>

</bean>
```

This makes the configuration easier to understand.

> Parameter-name based matching can depend on constructor parameter metadata being available at runtime/build time. For maximum explicitness, `index` or type information can be used when necessary.

---

# 21\. Setter Injection

Consider:

```
public class OrderService {

    private PaymentService paymentService;

    public void setPaymentService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }
}
```

XML:

```
<bean id="paymentService"
      class="in.strikes.PaymentService"/>

<bean id="orderService"
      class="in.strikes.OrderService">

    <property name="paymentService"
              ref="paymentService"/>

</bean>
```

---

# 22\. How `<property name="">` Works

Given:

```
<property name="paymentService"
          ref="paymentService"/>
```

Spring follows JavaBean property conventions.

Conceptually:

```
name = paymentService
          │
          ▼
setPaymentService(...)
```

Therefore, this Java method:

```
public void setPaymentService(
        PaymentService paymentService) {
}
```

matches:

```
<property name="paymentService"
          ref="paymentService"/>
```

---

## Another Example

Java:

```
public void setPaymentServiceBean(
        PaymentService paymentService) {
}
```

Then the property is:

```
<property name="paymentServiceBean"
          ref="paymentService"/>
```

The XML property name should correspond to the JavaBean property exposed by the setter.

---

# 23\. Constructor Injection vs Setter Injection

| Feature | Constructor Injection | Setter Injection |
| --- | --- | --- |
| XML tag | `<constructor-arg>` | `<property>` |
| Injection time | During object creation | After object creation |
| Required dependency | Excellent fit | Can be optional |
| Immutable fields | Better | Less suitable |
| Object validity | Can enforce required dependencies | Object can exist before dependency is set |
| Modern recommendation | Usually preferred for required dependencies | Useful for optional/configurable dependencies |

---

# 24\. Interface-Based Dependency Injection

Suppose:

```
public interface PaymentService {

    void pay();
}
```

Two implementations:

```
public class UpiPaymentService
        implements PaymentService {

    @Override
    public void pay() {
        System.out.println("UPI Payment");
    }
}
```

and:

```
public class CardPaymentService
        implements PaymentService {

    @Override
    public void pay() {
        System.out.println("Card Payment");
    }
}
```

`OrderService` depends on the interface:

```
public class OrderService {

    private PaymentService paymentService;

    public OrderService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }
}
```

---

# 25\. XML Resolves the Implementation Explicitly

XML:

```
<bean id="upiPaymentService"
      class="in.strikes.payment.UpiPaymentService"/>

<bean id="cardPaymentService"
      class="in.strikes.payment.CardPaymentService"/>

<bean id="orderService"
      class="in.strikes.OrderService">

    <constructor-arg ref="upiPaymentService"/>

</bean>
```

The important line is:

```
<constructor-arg ref="upiPaymentService"/>
```

Spring doesn't have to guess which implementation you want.

```
                   PaymentService
                        │
             ┌──────────┴──────────┐
             ▼                     ▼
      UpiPaymentService     CardPaymentService
             │
             │ ref
             ▼
        OrderService
```

To switch implementation:

```
<constructor-arg ref="cardPaymentService"/>
```

No Java source modification is required.

---

# 26\. Why XML Can Be Useful for Strategy Selection

Suppose production uses UPI:

```
<constructor-arg ref="upiPaymentService"/>
```

Testing might use another implementation:

```
<constructor-arg ref="mockPaymentService"/>
```

This allows configuration to decide which implementation is injected.

This was one of the important advantages of externalized Spring configuration.

---

# 27\. XML Bean Scopes

Spring supports multiple bean scopes, with:

```
singleton
prototype
```

being the most important basic Spring Core scopes.

---

# 28\. Singleton Scope

Default:

```
<bean id="userService"
      class="in.strikes.UserService"
      scope="singleton"/>
```

You can also omit the scope:

```
<bean id="userService"
      class="in.strikes.UserService"/>
```

because singleton is the default.

Conceptually:

```
getBean("userService")
        │
        ▼
   ┌───────────┐
   │ Singleton │
   │ Instance  │
   └───────────┘
        ▲
        │
   ┌────┴────┐
   │         │
Client A   Client B
```

Both clients receive the same instance.

---

# 29\. Prototype Scope

XML:

```
<bean id="userService"
      class="in.strikes.UserService"
      scope="prototype"/>
```

Every lookup creates a new instance:

```
UserService u1 =
        context.getBean("userService", UserService.class);

UserService u2 =
        context.getBean("userService", UserService.class);
```

Conceptually:

```
getBean()
   │
   ├──────► UserService #1
   │
   └──────► UserService #2
```

Therefore:

```
u1 == u2
```

is:

```
false
```

---

# 30\. Singleton vs Prototype

| Property | Singleton | Prototype |
| --- | --- | --- |
| Default | Yes | No |
| Number of instances | Usually one per container | New instance per retrieval |
| Creation | Eager by default for `ApplicationContext` | Created when requested |
| Destruction tracking | Spring manages destruction | Spring does not manage destruction after creation |
| Typical use | Stateless services | Stateful/request-specific objects |

---

# 31\. Important Prototype Lifecycle Rule

For a prototype bean:

```
<bean id="userService"
      class="in.strikes.UserService"
      scope="prototype"
      init-method="init"
      destroy-method="cleanUp"/>
```

Spring invokes initialization callbacks when the bean is created.

However, Spring does **not** track the prototype instance for normal container shutdown destruction.

Therefore:

```
Prototype creation
       │
       ▼
init-method
       │
       ▼
Bean returned
       │
       ▼
Application uses bean
       │
       ▼
Container shutdown
       │
       X
destroy-method NOT automatically invoked
```

If cleanup is required for prototype objects, the application must arrange it appropriately.

---

# 32\. Collection Injection

Spring XML provides dedicated tags for collection injection:

```
<list>
<set>
<map>
<props>
```

These can be used with constructor arguments or properties.

---

# 33\. Injecting a List

Java:

```
public class UserService {

    private List<String> userNames;

    public UserService(List<String> userNames) {
        this.userNames = userNames;
    }
}
```

XML:

```
<bean id="userService"
      class="in.strikes.UserService">

    <constructor-arg name="userNames">

        <list>
            <value>Aditya</value>
            <value>Rohit</value>
            <value>Rohan</value>
        </list>

    </constructor-arg>

</bean>
```

Conceptually:

```
             userNames
                 │
                 ▼
       ┌─────────────────┐
       │ List<String>    │
       ├─────────────────┤
       │ Aditya          │
       │ Rohit           │
       │ Rohan           │
       └─────────────────┘
```

---

# 34\. Injecting a Set

```
<constructor-arg name="userNames">

    <set>
        <value>Aditya</value>
        <value>Rohit</value>
        <value>Rohan</value>
    </set>

</constructor-arg>
```

A `Set` is useful when uniqueness is required.

```
XML values
   │
   ▼
 Set<String>
   │
   ├── Aditya
   ├── Rohit
   └── Rohan
```

---

# 35\. Injecting a Map

Java:

```
public UserService(Map<String, String> userMap) {
    this.userMap = userMap;
}
```

XML:

```
<bean id="userService"
      class="in.strikes.UserService">

    <constructor-arg name="userMap">

        <map>
            <entry key="101" value="Aditya"/>
            <entry key="102" value="Rohit"/>
        </map>

    </constructor-arg>

</bean>
```

Conceptually:

```
Map<String, String>

101 ─────► Aditya
102 ─────► Rohit
```

---

# 36\. Injecting Bean References into Collections

Collection elements don't have to be literals.

Suppose:

```
<bean id="upiPaymentService"
      class="in.strikes.payment.UpiPaymentService"/>

<bean id="cardPaymentService"
      class="in.strikes.payment.CardPaymentService"/>
```

A list can contain bean references:

```
<list>
    <ref bean="upiPaymentService"/>
    <ref bean="cardPaymentService"/>
</list>
```

This is different from:

```
<value>upiPaymentService</value>
```

because:

```
<value>
    literal value
</value>

<ref>
    reference to another Spring bean
</ref>
```

---

# 37\. Bean Lifecycle

A simplified Spring bean lifecycle looks like:

```
XML Configuration
       │
       ▼
BeanDefinition
       │
       ▼
Bean Instantiation
       │
       ▼
Dependency Injection
       │
       ▼
Bean Initialization
       │
       ▼
Bean Ready
       │
       ▼
Application Uses Bean
       │
       ▼
Container Shutdown
       │
       ▼
Bean Destruction
```

For XML configuration, lifecycle methods can be declared directly.

---

# 38\. `init-method`

Java:

```
public class UserService {

    public void init() {
        System.out.println("UserService initialized");
    }
}
```

XML:

```
<bean id="userService"
      class="in.strikes.UserService"
      init-method="init"/>
```

Spring calls:

```
userService.init();
```

at the appropriate initialization phase.

---

# 39\. `destroy-method`

Java:

```
public void cleanUp() {
    System.out.println("UserService destroyed");
}
```

XML:

```
<bean id="userService"
      class="in.strikes.UserService"
      destroy-method="cleanUp"/>
```

For a singleton bean, Spring invokes the configured destroy method when the application context is closed.

---

# 40\. Complete Lifecycle Example

Java:

```
public class UserService {

    public UserService() {
        System.out.println("Constructor");
    }

    public void init() {
        System.out.println("Init method");
    }

    public void cleanUp() {
        System.out.println("Destroy method");
    }
}
```

XML:

```
<bean id="userService"
      class="in.strikes.UserService"
      init-method="init"
      destroy-method="cleanUp"/>
```

Main:

```
ClassPathXmlApplicationContext context =
        new ClassPathXmlApplicationContext("beans.xml");

context.close();
```

Typical lifecycle:

```
ApplicationContext created
        │
        ▼
Constructor
        │
        ▼
Dependency Injection
        │
        ▼
init()
        │
        ▼
Bean Ready
        │
        ▼
context.close()
        │
        ▼
cleanUp()
```

---

# 41\. XML Lifecycle vs Annotation Lifecycle

| XML | Annotation |
| --- | --- |
| `init-method` | `@PostConstruct` |
| `destroy-method` | `@PreDestroy` |

Example XML:

```
<bean id="userService"
      class="in.strikes.UserService"
      init-method="init"
      destroy-method="cleanUp"/>
```

Equivalent annotation style:

```
@PostConstruct
public void init() {
}

@PreDestroy
public void cleanUp() {
}
```

---

# 42\. Closing the ApplicationContext

Use:

```
ClassPathXmlApplicationContext context =
        new ClassPathXmlApplicationContext("beans.xml");

context.close();
```

Calling `close()` is important when you want Spring to perform destruction callbacks for managed singleton beans.

A useful alternative is try-with-resources where supported:

```
try (ClassPathXmlApplicationContext context =
         new ClassPathXmlApplicationContext("beans.xml")) {

    // application logic
}
```

This ensures the context is closed automatically.

---

# 43\. Multiple XML Configuration Files

A large XML file can become difficult to maintain.

Instead of:

```
beans.xml
```

containing hundreds or thousands of definitions, divide configuration logically:

```
beans.xml
database.xml
security.xml
services.xml
```

Example:

```
resources/
├── appConfig.xml
├── beans.xml
├── database.xml
└── services.xml
```

---

# 44\. `<import>` Tag

A main configuration file can import others.

```
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="
           http://www.springframework.org/schema/beans
           http://www.springframework.org/schema/beans/spring-beans.xsd">

    <import resource="beans.xml"/>
    <import resource="database.xml"/>
    <import resource="services.xml"/>

</beans>
```

Application:

```
ApplicationContext context =
        new ClassPathXmlApplicationContext("appConfig.xml");
```

---

# 45\. Modular XML Architecture

```
                    appConfig.xml
                         │
          ┌──────────────┼──────────────┐
          │              │              │
          ▼              ▼              ▼
      beans.xml     database.xml    services.xml
          │              │              │
          ▼              ▼              ▼
       Beans        DB Config       Services
          │              │              │
          └──────────────┼──────────────┘
                         ▼
                    IoC Container
```

Benefits:

- Easier maintenance
- Better organization
- Separation of concerns
- Easier migration
- Smaller configuration files

---

# 46\. Hybrid XML \+ Annotation Configuration

Legacy applications often contain both:

```
XML beans
+
@Component
+
@Service
+
@Repository
```

Spring can support both approaches in the same application context.

Enable component scanning:

```
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"

       xsi:schemaLocation="
           http://www.springframework.org/schema/beans
           http://www.springframework.org/schema/beans/spring-beans.xsd

           http://www.springframework.org/schema/context
           http://www.springframework.org/schema/context/spring-context.xsd">

    <context:component-scan base-package="in.strikes"/>

</beans>
```

---

# 47\. How Hybrid Configuration Works

Suppose:

```
@Service
public class OrderService {
}
```

and XML contains:

```
<bean id="paymentService"
      class="in.strikes.PaymentService"/>
```

Component scanning finds:

```
@Service
public class OrderService
```

XML parsing finds:

```
<bean id="paymentService".../>
```

Both become Spring-managed beans.

```
                       Spring IoC
                           │
             ┌─────────────┴─────────────┐
             │                           │
             ▼                           ▼
       XML Configuration          Component Scanning
             │                           │
             ▼                           ▼
      paymentService               OrderService
             │                           │
             └─────────────┬─────────────┘
                           ▼
                    Same ApplicationContext
```

---

# 48\. Why Hybrid Configuration Is Important

A common migration strategy is:

```
Old Application
      │
      ▼
XML-heavy Spring Application
      │
      ▼
Introduce annotations
      │
      ▼
XML + Annotation Hybrid
      │
      ▼
Gradually migrate
      │
      ▼
Java/Annotation Configuration
      │
      ▼
Spring Boot
```

This allows teams to migrate incrementally rather than rewriting the entire application at once.

---

# 49\. Complete Example

Let's create a small application.

## `PaymentService.java`

```
package in.strikes;

public class PaymentService {

    public void pay() {
        System.out.println("Payment processed");
    }
}
```

## `OrderService.java`

```
package in.strikes;

public class OrderService {

    private final PaymentService paymentService;

    public OrderService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }

    public void placeOrder() {

        paymentService.pay();

        System.out.println("Order placed");
    }
}
```

---

## `beans.xml`

```
<?xml version="1.0" encoding="UTF-8"?>

<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="
           http://www.springframework.org/schema/beans
           http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="paymentService"
          class="in.strikes.PaymentService"/>

    <bean id="orderService"
          class="in.strikes.OrderService">

        <constructor-arg ref="paymentService"/>

    </bean>

</beans>
```

---

## `Main.java`

```
package in.strikes;

import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class Main {

    public static void main(String[] args) {

        ApplicationContext context =
                new ClassPathXmlApplicationContext("beans.xml");

        OrderService orderService =
                context.getBean("orderService",
                                OrderService.class);

        orderService.placeOrder();
    }
}
```

Output:

```
Payment processed
Order placed
```

---

# 50\. Complete Dependency Diagram

```
                 beans.xml
                     │
                     ▼
          ClassPathXmlApplicationContext
                     │
                     ▼
              Bean Definitions
                     │
          ┌──────────┴──────────┐
          │                     │
          ▼                     ▼
   PaymentService          OrderService
          │                     │
          │                     │
          └────── dependency ───┘
                     │
                     ▼
          constructor injection
                     │
                     ▼
              IoC Container
```

---

# 51\. Important XML Tags Cheat Sheet

| Tag/Attribute | Purpose |
| --- | --- |
| `<beans>` | Root configuration element |
| `<bean>` | Declares a Spring bean |
| `id` | Primary bean identifier |
| `name` | Bean aliases/names |
| `class` | Fully qualified Java class |
| `scope` | Bean scope |
| `<constructor-arg>` | Constructor injection |
| `ref` | Reference another Spring bean |
| `value` | Literal value |
| `index` | Constructor argument position |
| `name` | Constructor/property name |
| `<property>` | Setter/property injection |
| `<list>` | List injection |
| `<set>` | Set injection |
| `<map>` | Map injection |
| `<entry>` | Map entry |
| `<import>` | Import another XML configuration |
| `init-method` | Initialization callback |
| `destroy-method` | Destruction callback |
| `<context:component-scan>` | Enable annotation component scanning |

---

# 52\. XML Configuration Mental Model

A very useful way to remember Spring XML is:

```
<bean>
   │
   ├── id
   │
   ├── class
   │
   ├── scope
   │
   ├── constructor-arg
   │       │
   │       ├── ref
   │       ├── value
   │       ├── index
   │       └── name
   │
   ├── property
   │       │
   │       ├── ref
   │       └── value
   │
   ├── init-method
   │
   └── destroy-method
```

If you understand this structure, most basic Spring XML configurations become easy to read.

---

# 53\. Common Mistakes

## Mistake 1 — Wrong XML Location

Incorrect:

```
src/main/java/beans.xml
```

Usually place it under:

```
src/main/resources/beans.xml
```

---

## Mistake 2 — Wrong Class Name

Incorrect:

```
class="OrderService"
```

Correct:

```
class="in.strikes.OrderService"
```

unless another supported configuration mechanism resolves it differently.

---

## Mistake 3 — Using `value` Instead of `ref`

Incorrect:

```
<constructor-arg value="paymentService"/>
```

This attempts to provide the literal value:

```
"paymentService"
```

Correct:

```
<constructor-arg ref="paymentService"/>
```

This injects the Spring bean.

---

## Mistake 4 — Assuming Type Lookup Always Works

This:

```
context.getBean(PaymentService.class);
```

fails if multiple suitable beans exist and none is uniquely resolvable.

When you know the desired bean:

```
context.getBean("upiPaymentService",
                PaymentService.class);
```

---

## Mistake 5 — Forgetting to Close the Context

For applications where lifecycle cleanup matters:

```
context.close();
```

or use try-with-resources.

---

# 54\. Interview Questions

## Beginner Level

### Q1. What is XML-based configuration in Spring?

**Answer:**

XML-based configuration is a method of providing Spring IoC container metadata using an XML file instead of annotations or Java configuration.

Example:

```
<bean id="orderService"
      class="in.strikes.OrderService"/>
```

Spring reads this metadata and creates/manages the bean.

---

### Q2. Which class is commonly used to load XML configuration from the classpath?

**Answer:**

```
ClassPathXmlApplicationContext
```

Example:

```
ApplicationContext context =
        new ClassPathXmlApplicationContext("beans.xml");
```

---

### Q3. Where should `beans.xml` normally be placed in a Maven project?

**Answer:**

```
src/main/resources/
```

For example:

```
src/main/resources/beans.xml
```

Maven places resources on the application classpath.

---

### Q4. What is the purpose of the `<bean>` tag?

**Answer:**

It declares a Spring-managed bean.

Example:

```
<bean id="userService"
      class="in.strikes.UserService"/>
```

---

### Q5. What is the purpose of the `id` attribute?

**Answer:**

It gives the bean an identifier/name that can be used to retrieve or reference the bean.

```
<bean id="userService"
      class="in.strikes.UserService"/>
```

---

### Q6. What does the `class` attribute represent?

**Answer:**

It represents the fully qualified class name of the object Spring should instantiate.

```
class="in.strikes.UserService"
```

---

### Q7. What is the default scope of a Spring bean?

**Answer:**

The default scope is:

```
singleton
```

---

### Q8. What is `ref` used for?

**Answer:**

`ref` references another Spring-managed bean.

```
<constructor-arg ref="paymentService"/>
```

---

### Q9. What is `value` used for?

**Answer:**

`value` supplies a literal configuration value.

```
<constructor-arg value="UPI"/>
```

---

### Q10. What is the difference between `ref` and `value`?

**Answer:**

```
ref   → another Spring bean
value → literal value
```

Example:

```
<constructor-arg ref="paymentService"/>
<constructor-arg value="UPI"/>
```

---

# 55\. Intermediate Interview Questions

## Q11. What are the three common ways to call `getBean()`?

**Answer:**

### By name

```
context.getBean("orderService");
```

### By type

```
context.getBean(OrderService.class);
```

### By name and type

```
context.getBean("orderService",
                OrderService.class);
```

---

## Q12. Why is name + type lookup often preferable?

**Answer:**

Because it avoids explicit casting while identifying the desired bean by name.

```
OrderService service =
        context.getBean("orderService",
                        OrderService.class);
```

---

## Q13. What happens when `getBean(OrderService.class)` finds two beans?

**Answer:**

If both beans are suitable candidates and Spring cannot choose one, it throws:

```
NoUniqueBeanDefinitionException
```

---

## Q14. What happens if the requested bean name does not exist?

**Answer:**

Spring typically throws:

```
NoSuchBeanDefinitionException
```

---

## Q15. What is constructor injection in XML?

**Answer:**

Constructor injection supplies dependencies through `<constructor-arg>`.

Example:

```
<bean id="orderService"
      class="in.strikes.OrderService">

    <constructor-arg ref="paymentService"/>

</bean>
```

---

## Q16. What is setter injection in XML?

**Answer:**

Setter injection supplies dependencies through JavaBean setter methods using `<property>`.

```
<property name="paymentService"
          ref="paymentService"/>
```

---

## Q17. How does Spring map `<property name="paymentService">` to a setter?

**Answer:**

Through JavaBean property conventions.

```
name="paymentService"
```

corresponds conceptually to:

```
setPaymentService(...)
```

---

## Q18. Can XML configuration perform field injection?

**Answer:**

Traditional XML bean configuration does not directly express field injection like:

```
@Autowired
private PaymentService paymentService;
```

XML DI is normally expressed through constructors and setter properties.

---

## Q19. How do you configure prototype scope?

**Answer:**

```
<bean id="userService"
      class="in.strikes.UserService"
      scope="prototype"/>
```

---

## Q20. What happens when you request a prototype bean twice?

**Answer:**

Spring creates two different instances.

```
UserService u1 =
        context.getBean(UserService.class);

UserService u2 =
        context.getBean(UserService.class);
```

For an unambiguous prototype bean:

```
u1 != u2
```

---

# 56\. Advanced Interview Questions

## Q21. Why does XML resolve interface ambiguity using `ref`?

Consider:

```
PaymentService
       ▲
       │
 ┌─────┴─────┐
 │           │
UPI         Card
```

XML can explicitly say:

```
<constructor-arg ref="upiPaymentService"/>
```

Therefore Spring doesn't have to select between all implementations solely by type.

---

## Q22. What is the purpose of `<import>`?

**Answer:**

It allows one XML configuration file to include definitions from other XML files.

```
<import resource="database.xml"/>
<import resource="services.xml"/>
```

This supports modular configuration.

---

## Q23. How can XML and annotations be used together?

**Answer:**

Enable component scanning:

```
<context:component-scan
        base-package="in.strikes"/>
```

Then XML `<bean>` definitions and annotation-based components can coexist in the same application context.

---

## Q24. What is `init-method`?

**Answer:**

It specifies a method Spring should invoke during bean initialization.

```
<bean id="userService"
      class="in.strikes.UserService"
      init-method="init"/>
```

---

## Q25. What is `destroy-method`?

**Answer:**

It specifies a method Spring should invoke during destruction of an eligible managed bean, particularly singleton beans when the context is closed.

```
<bean id="userService"
      class="in.strikes.UserService"
      destroy-method="cleanUp"/>
```

---

## Q26. Does Spring call `destroy-method` for prototype beans?

**Answer:**

No.

Spring creates and initializes prototype objects but does not normally manage their destruction after handing them to the caller.

---

## Q27. What is the difference between `scope="singleton"` and `scope="prototype"`?

**Answer:**

```
singleton
   │
   └── one shared instance per Spring container

prototype
   │
   └── new instance for each bean retrieval
```

---

## Q28. Can a collection contain references to other beans?

**Answer:**

Yes.

Example:

```
<list>
    <ref bean="upiPaymentService"/>
    <ref bean="cardPaymentService"/>
</list>
```

---

## Q29. What is the difference between `<list>` and `<set>`?

**Answer:**

A `List` represents an ordered collection that can contain duplicates, while a `Set` represents a collection whose semantics enforce uniqueness.

---

## Q30. What is the purpose of `<map>`?

**Answer:**

It is used to configure key-value pairs.

```
<map>
    <entry key="101" value="Aditya"/>
    <entry key="102" value="Rohit"/>
</map>
```

---

# 57\. Scenario-Based Interview Questions

## Q31. You have two `PaymentService` implementations. How would you inject UPI using XML?

```
<bean id="upiPaymentService"
      class="in.strikes.UpiPaymentService"/>

<bean id="cardPaymentService"
      class="in.strikes.CardPaymentService"/>

<bean id="orderService"
      class="in.strikes.OrderService">

    <constructor-arg ref="upiPaymentService"/>

</bean>
```

---

## Q32. How would you switch from UPI to Card without changing Java code?

Change:

```
<constructor-arg ref="upiPaymentService"/>
```

to:

```
<constructor-arg ref="cardPaymentService"/>
```

This is a good example of externalized dependency configuration.

---

## Q33. Your XML contains:

```
<constructor-arg value="paymentService"/>
```

but the constructor expects `PaymentService`. What is wrong?

`value` represents a literal value.

The correct configuration is:

```
<constructor-arg ref="paymentService"/>
```

---

## Q34. Two beans have the same class but different IDs. What happens with this?

```
context.getBean(OrderService.class);
```

If both are candidates, Spring cannot uniquely determine the requested bean and throws:

```
NoUniqueBeanDefinitionException
```

Use:

```
context.getBean("orderService1",
                OrderService.class);
```

to identify the required bean.

---

## Q35. A bean has `destroy-method="cleanUp"`, but cleanup never runs. What should you check?

Check:

1. Is the bean a singleton?
2. Was the `ApplicationContext` closed?
3. Is the destroy method name correct?
4. Is the method accessible and valid for Spring's destruction callback mechanism?
5. Is the bean actually managed by that context?

For example:

```
context.close();
```

should be called when manually managing the application context.

---

# 58\. Frequently Asked Trick Questions

## Q36. Is XML configuration completely obsolete?

**Answer:**

No.

It is less common in new Spring Boot applications, but it remains relevant for:

- Legacy applications
- Enterprise systems
- Migration projects
- Third-party XML configuration
- Hybrid Spring applications

---

## Q37. Does XML configuration mean Spring is not using IoC?

**Answer:**

No.

XML is simply a **configuration metadata format**.

Spring still performs:

```
IoC
DI
Bean lifecycle management
Scope management
```

---

## Q38. Does XML create objects outside Spring?

**Answer:**

No.

The `<bean>` definition tells Spring how to create and manage the object.

---

## Q39. Does `ref` create a new object?

**Answer:**

No.

It refers to another Spring-managed bean.

---

## Q40. Is `getBean(Class)` always better than `getBean(String)`?

**Answer:**

Not necessarily.

Type-based lookup is convenient and type-safe, but it can become ambiguous when multiple beans of that type exist.

A good explicit choice is often:

```
context.getBean("beanName", BeanType.class);
```

---

# 59\. Quick Revision Diagram

```
                     SPRING XML CONFIGURATION
                              │
                              ▼
                         beans.xml
                              │
                              ▼
                         <bean>
                              │
          ┌───────────────────┼───────────────────┐
          │                   │                   │
          ▼                   ▼                   ▼
         id                 class               scope
          │                   │                   │
          │                   │             ┌─────┴─────┐
          │                   │             │           │
          │                   │         singleton    prototype
          │                   │
          └──────────┬────────┘
                     ▼
              BeanDefinition
                     │
                     ▼
             Dependency Injection
                     │
             ┌───────┴────────┐
             ▼                ▼
       Constructor          Setter
       <constructor-arg>    <property>
             │                │
             ├── ref          ├── ref
             ├── value        └── value
             ├── index
             └── name
                     │
                     ▼
                 Lifecycle
                     │
             ┌───────┴────────┐
             ▼                ▼
       init-method       destroy-method
```

---

# 60\. One-Page Cheat Sheet

## Container

```
ApplicationContext context =
        new ClassPathXmlApplicationContext("beans.xml");
```

## Bean

```
<bean id="orderService"
      class="in.strikes.OrderService"/>
```

## Constructor DI

```
<constructor-arg ref="paymentService"/>
```

## Literal value

```
<constructor-arg value="UPI"/>
```

## Setter DI

```
<property name="paymentService"
          ref="paymentService"/>
```

## Singleton

```
scope="singleton"
```

## Prototype

```
scope="prototype"
```

## Init

```
init-method="init"
```

## Destroy

```
destroy-method="cleanUp"
```

## List

```
<list>
    <value>A</value>
    <value>B</value>
</list>
```

## Set

```
<set>
    <value>A</value>
    <value>B</value>
</set>
```

## Map

```
<map>
    <entry key="101" value="Aditya"/>
</map>
```

## Import

```
<import resource="services.xml"/>
```

## Component Scan

```
<context:component-scan
        base-package="in.strikes"/>
```

## Get bean

```
context.getBean("orderService");
```

```
context.getBean(OrderService.class);
```

```
context.getBean("orderService",
                OrderService.class);
```

---

# 61\. Final Interview Summary

Remember these **10 points**:

1. **XML is configuration metadata**, not a replacement for IoC.
2. `ClassPathXmlApplicationContext` loads XML configuration from the classpath.
3. Maven resources normally go under `src/main/resources`.
4. `<bean>` declares a Spring-managed bean.
5. `ref` means **another Spring bean**; `value` means **literal value**.
6. `<constructor-arg>` is used for constructor injection.
7. `<property>` is used for setter/property injection.
8. `singleton` is the default scope; `prototype` creates a new instance per lookup.
9. `init-method` and `destroy-method` configure lifecycle callbacks.
10. XML and annotations can coexist using `<context:component-scan>`.

### The core idea to remember

```
              XML Configuration
                     │
                     ▼
              Bean Definitions
                     │
                     ▼
                IoC Container
                     │
          ┌──────────┼──────────┐
          ▼          ▼          ▼
      Creation       DI      Lifecycle
          │          │          │
          └──────────┼──────────┘
                     ▼
             Managed Objects
```

> **Interview-ready definition:**\
>  **Spring XML-based configuration is a declarative way of supplying bean metadata to the Spring IoC container. Using elements such as `<bean>`, `<constructor-arg>`, and `<property>`, we can define objects, their dependencies, scopes, and lifecycle callbacks without putting Spring configuration annotations directly into the Java classes.**
