# Spring Framework & Client-Server Architecture

> **Goal:** Understand how a request travels from a client to a Spring Boot application, how Spring is built on top of Servlets, how databases fit into the stack, and how monolithic applications differ from microservices.

---

# 1\. Client-Server Architecture

## 1.1 What is Client-Server Architecture?

Client-server architecture is a model in which:

- **Client** sends a request.
- **Server** receives and processes the request.
- **Server** sends a response back.

### Basic Flow

```
+------------------+                         +------------------+
|                  |       HTTP Request      |                  |
|     CLIENT       | ----------------------> |      SERVER      |
|                  |                         |                  |
| Browser / Mobile |                         | Spring Boot App  |
| Postman / React  |       HTTP Response     | Business Logic   |
| Another Server   | <---------------------- | Database         |
|                  |                         |                  |
+------------------+                         +------------------+
```

### Example

Suppose a user opens:

```
https://example.com/courses
```

The browser acts as the **client**.

It sends:

```
GET /courses
```

The server:

1. Receives the request.
2. Determines that `/courses` is being requested.
3. Executes application logic.
4. May query a database.
5. Creates a response.
6. Sends it back to the browser.

---

# 2\. What Can Be a Client?

A client is simply something that **initiates a request**.

Common examples:

```
Client
 ├── Web Browser
 ├── Mobile Application
 ├── React / Angular / Vue Frontend
 ├── Postman
 ├── Another Backend Server
 └── Any HTTP-capable application
```

### Important Interview Point

> **Client does NOT necessarily mean browser.**

For example:

```
Order Service ─────HTTP─────> Payment Service
```

Here:

- Order Service = Client
- Payment Service = Server

The same application can therefore behave as a **client in one interaction** and a **server in another**.

---

# 3\. What Does a Server Do?

A server is a system that listens for requests and provides responses.

A backend server may:

- Retrieve data
- Store data
- Update data
- Delete data
- Authenticate users
- Authorize users
- Perform calculations
- Communicate with other services
- Communicate with databases

Example:

```
Client
   |
   | GET /users/101
   v
Spring Boot Server
   |
   | Query
   v
Database
   |
   | User data
   v
Spring Boot Server
   |
   | JSON response
   v
Client
```

---

# 4\. HTTP — The Communication Protocol

## 4.1 What is HTTP?

**HTTP = Hypertext Transfer Protocol**

HTTP defines rules for communication between clients and servers.

It defines things such as:

- Request methods
- Headers
- Status codes
- Request/response structure

Example:

```
Client                         Server

GET /courses  -------------------->

             <-------------------- 200 OK
```

---

# 5\. HTTP Methods

HTTP methods indicate what operation the client wants to perform.

| Method | Common Purpose |
| --- | --- |
| GET | Retrieve data |
| POST | Create/submit data |
| PUT | Replace/update a resource |
| PATCH | Partially update a resource |
| DELETE | Delete a resource |

---

## 5.1 GET

Used to retrieve information.

```
GET /courses
```

Example response:

```
[
  {
    "id": 1,
    "name": "Java"
  },
  {
    "id": 2,
    "name": "Spring Boot"
  }
]
```

---

## 5.2 POST

Usually used to create a new resource.

```
POST /courses
```

Request body:

```
{
  "name": "Spring Boot",
  "price": 999
}
```

Possible response:

```
201 Created
```

---

## 5.3 PUT

Usually used to replace the representation of an existing resource.

```
PUT /courses/10
```

```
{
  "name": "Advanced Spring Boot",
  "price": 1499
}
```

Think:

```
PUT = Replace/update the complete resource
```

---

## 5.4 PATCH

Used for partial modification.

For example, if you only want to change the price:

```
PATCH /courses/10
```

```
{
  "price": 1299
}
```

Think:

```
PATCH = Change only part of a resource
```

---

## 5.5 DELETE

Used to delete a resource.

```
DELETE /courses/10
```

Possible response:

```
204 No Content
```

---

# 6\. HTTP vs HTTPS

## HTTP

HTTP transfers data without transport encryption.

Conceptually:

```
Client
   |
   |  Data
   |---------------->
   |
   v
Network
```

An attacker who can observe the traffic may be able to read sensitive information.

---

## HTTPS

HTTPS = HTTP + TLS security.

```
Client
   |
   | Encrypted HTTPS traffic
   |=========================>
   |
 Server
```

HTTPS provides protection for data in transit through encryption and also provides server authentication via certificates.

### Interview Answer

> HTTPS is the secure version of HTTP that uses TLS to protect communication between client and server.

### Important

Do **not** say:

> HTTPS encrypts the entire internet.

More accurately:

> HTTPS encrypts HTTP communication between the client and server at the transport connection.

---

# 7\. Anatomy of an HTTP Request

An HTTP request commonly contains:

1. Method
2. Target/path
3. Headers
4. Optional body

Example:

```
POST /login HTTP/1.1
Host: example.com
Content-Type: application/json
Accept: application/json

{
    "email": "user@example.com",
    "password": "secret"
}
```

Diagram:

```
+--------------------------------------+
| METHOD + REQUEST TARGET              |
| POST /login                          |
+--------------------------------------+
| HEADERS                              |
| Host: example.com                    |
| Content-Type: application/json       |
| Accept: application/json             |
+--------------------------------------+
| BODY                                 |
| {                                    |
|   "email": "user@example.com"        |
|   "password": "secret"               |
| }                                    |
+--------------------------------------+
```

---

# 8\. HTTP Headers

Headers provide metadata.

Examples:

```
Content-Type: application/json
Accept: application/json
Authorization: Bearer <token>
User-Agent: Mozilla/5.0
```

## Content-Type

Tells the server what format the request body uses.

Example:

```
Content-Type: application/json
```

means:

> The request body contains JSON.

---

## Accept

Tells the server what response representation the client prefers.

```
Accept: application/json
```

means:

> I prefer JSON in the response.

### Interview Difference

```
Content-Type
    ↓
What format am I SENDING?

Accept
    ↓
What format do I WANT BACK?
```

---

# 9\. Request Body

The request body contains data sent to the server.

Example:

```
{
  "name": "Rahul",
  "email": "rahul@example.com"
}
```

A body is commonly used with:

```
POST
PUT
PATCH
```

A `GET` request generally does not use a request body in typical API design.

---

# 10\. Anatomy of an HTTP Response

An HTTP response contains:

1. Status code
2. Headers
3. Optional body

Example:

```
HTTP/1.1 200 OK
Content-Type: application/json

{
    "message": "Login successful"
}
```

Diagram:

```
+--------------------------------------+
| STATUS                               |
| 200 OK                               |
+--------------------------------------+
| HEADERS                              |
| Content-Type: application/json       |
+--------------------------------------+
| BODY                                 |
| {                                    |
|   "message": "Login successful"      |
| }                                    |
+--------------------------------------+
```

---

# 11\. HTTP Status Codes

Status codes are grouped into categories.

| Range | Meaning |
| --- | --- |
| 1xx | Informational |
| 2xx | Success |
| 3xx | Redirection |
| 4xx | Client-side error |
| 5xx | Server-side error |

---

## Important Status Codes

### 200 OK

Request succeeded.

```
GET /courses

200 OK
```

---

### 201 Created

A new resource was successfully created.

```
POST /courses

201 Created
```

---

### 204 No Content

Request succeeded but there is no response body.

Common example:

```
DELETE /courses/10

204 No Content
```

---

### 400 Bad Request

The server cannot process the request because the request is invalid.

---

### 401 Unauthorized

Authentication is required or failed.

Think:

```
Who are you?
```

---

### 403 Forbidden

The server understands who you are but you don't have permission.

Think:

```
I know who you are,
but you are not allowed to do this.
```

---

### 404 Not Found

Requested resource cannot be found.

```
GET /courses/999999

404 Not Found
```

---

### 500 Internal Server Error

Something unexpected went wrong on the server.

---

### 503 Service Unavailable

The server/service is currently unable to handle the request, often because of temporary overload or maintenance.

---

# 12\. Core Java Application vs Web Server

This is one of the most important conceptual differences.

## Normal Java Program

Suppose:

```
public class Main {
    public static void main(String[] args) {
        System.out.println("Hello");
    }
}
```

Execution:

```
Compile
   |
   v
Main.class
   |
   v
JVM
   |
   v
main()
   |
   v
Program finishes
```

The program has a finite lifecycle.

```
START
  |
  v
EXECUTE
  |
  v
STOP
  |
  v
EXIT
```

---

# 13\. Web Server Program

A web server generally needs to stay available.

Conceptually:

```
START
  |
  v
START SERVER
  |
  v
WAIT FOR REQUEST
  |
  v
PROCESS REQUEST
  |
  v
SEND RESPONSE
  |
  +------------------+
                     |
                     v
               WAIT AGAIN
```

Conceptually this resembles:

```
while (true) {
    Request request = waitForRequest();
    Response response = process(request);
    sendResponse(response);
}
```

### Important

This is a **conceptual model**, not how modern servers necessarily implement concurrency internally.

---

# 14\. How Does a Computer Know Which Application Gets a Request?

The answer involves:

- IP address
- Port number

## IP Address

Identifies a machine/network endpoint.

Example:

```
127.0.0.1
```

is the IPv4 loopback address.

---

## Port

Identifies a network service/application endpoint on that host.

Example:

```
127.0.0.1:8080
```

Here:

```
127.0.0.1 = host
8080      = port
```

---

# 15\. IP + Port Analogy

Imagine an apartment building.

```
IP Address = Building Address

Port = Apartment Number
```

Example:

```
127.0.0.1:8080
```

means:

```
Building: 127.0.0.1
Apartment: 8080
```

This is only an analogy; networking ports identify service endpoints rather than literally identifying an application process.

---

# 16\. ServerSocket

Java's low-level networking API provides:

```
ServerSocket serverSocket = new ServerSocket(8080);
```

This tells the program to listen for TCP connections on port `8080`.

Conceptually:

```
Client
   |
   | TCP connection
   v
127.0.0.1:8080
   |
   v
ServerSocket
```

---

# 17\. Low-Level Java Networking

Java provides networking APIs in:

```
java.net
```

For example:

```
ServerSocket
Socket
```

These allow developers to build network applications.

But building a complete HTTP server manually is complicated.

---

# 18\. Why Building an HTTP Server with Raw Sockets Is Difficult

## Problem 1 — Raw Data

The socket gives you network data.

You need to read and interpret it.

```
Network
   |
   v
Raw bytes
   |
   v
Characters/text
```

---

## Problem 2 — HTTP Parsing

You must understand things like:

```
GET /courses HTTP/1.1
Host: example.com
Accept: application/json
```

You would have to parse:

```
Method     = GET
Path       = /courses
Version    = HTTP/1.1
Headers    = ...
Body       = ...
```

---

## Problem 3 — Routing

You would manually write logic such as:

```
if (endpoint.equals("/courses")) {
    getCourses();
} else if (endpoint.equals("/users")) {
    getUsers();
} else if (endpoint.equals("/login")) {
    login();
}
```

This becomes difficult as the application grows.

---

## Problem 4 — Response Construction

You must manually construct valid HTTP responses.

Conceptually:

```
HTTP/1.1 200 OK
Content-Type: application/json
Content-Length: ...

{...JSON...}
```

---

## Problem 5 — Concurrency

Suppose 1,000 users send requests simultaneously.

A server needs to handle multiple connections concurrently.

You need to deal with:

```
Threading
Thread pools
Blocking
Resource management
Connection handling
```

This is a major reason server/container frameworks exist.

---

# 19\. Servlets

The Servlet technology was introduced to simplify server-side Java web development.

## What is a Servlet?

A Servlet is a Java component designed to handle requests and generate responses in a servlet-based web environment.

Conceptually:

```
HTTP Request
     |
     v
Servlet Container
     |
     v
Servlet
     |
     v
Application Logic
     |
     v
HTTP Response
```

---

# 20\. Servlet Container

A **Servlet Container** manages the servlet environment.

Examples include:

- Apache Tomcat
- Jetty
- Undertow

A servlet container handles infrastructure responsibilities such as:

- Accepting connections
- HTTP request processing
- Servlet lifecycle
- Mapping requests to servlets
- Managing request processing
- Producing HTTP responses
- Concurrency/resource management

---

# 21\. Tomcat

Apache Tomcat is one of the most commonly used Servlet containers.

A simplified architecture:

```
                 HTTP Request
                      |
                      v
              +---------------+
              |    TOMCAT     |
              |               |
              | Servlet       |
              | Container     |
              +-------+-------+
                      |
                      v
               +-------------+
               |   Servlet   |
               +-------------+
                      |
                      v
                Application
                      |
                      v
                 HTTP Response
```

---

# 22\. HttpServletRequest and HttpServletResponse

The Servlet API gives Java code structured representations of the request and response.

### Request

```
HttpServletRequest request
```

Contains information such as:

```
HTTP method
Request URI
Headers
Parameters
Body
```

### Response

```
HttpServletResponse response
```

Allows application code to control things such as:

```
Status code
Response headers
Response body
```

---

# 23\. Servlet Lifecycle

A simplified servlet lifecycle is:

```
Servlet class
     |
     v
Container creates servlet
     |
     v
init()
     |
     v
service()
     |
     v
destroy()
```

For an `HttpServlet`, the `service()` processing can dispatch requests to methods such as:

```
doGet()
doPost()
doPut()
doDelete()
```

Example:

```
public class CourseServlet extends HttpServlet {

    @Override
    protected void doGet(
            HttpServletRequest request,
            HttpServletResponse response) {

        // Handle GET request
    }
}
```

---

# 24\. The Big Problem: Boilerplate

Servlets solved many networking problems.

But developers still had to deal with framework/infrastructure concerns.

As applications became larger, another problem became important:

> **How do we manage the relationships between hundreds or thousands of objects?**

This leads us to Spring.

---

# 25\. Spring Framework

Spring is a large Java ecosystem/framework designed to simplify enterprise application development.

One of its central ideas is:

> **Loose coupling through Inversion of Control and Dependency Injection.**

---

# 26\. Tight Coupling vs Loose Coupling

## Tight Coupling

Suppose:

```
class OrderService {

    private PaymentService paymentService =
            new PaymentService();

}
```

`OrderService` directly creates its dependency.

```
OrderService
     |
     | creates
     v
PaymentService
```

Changing the implementation can become difficult.

---

## Loose Coupling

Instead, provide the dependency from outside:

```
class OrderService {

    private PaymentService paymentService;

    OrderService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }
}
```

Now:

```
PaymentService
      |
      v
OrderService
```

The `OrderService` doesn't decide how its dependency is created.

This is the basic idea behind Dependency Injection.

---

# 27\. Inversion of Control (IoC)

Normally, your code controls object creation:

```
PaymentService paymentService =
        new PaymentService();
```

With Spring:

```
Spring Container
      |
      | creates/manages
      v
PaymentService
      |
      | injected into
      v
OrderService
```

The control over object creation and dependency management is moved to the framework/container.

That's the basic idea of **Inversion of Control**.

---

# 28\. Dependency Injection (DI)

Dependency Injection is a technique for implementing IoC.

Example:

```
@Service
class PaymentService {
}
```

```
@Service
class OrderService {

    private final PaymentService paymentService;

    public OrderService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }
}
```

Spring creates `PaymentService` and provides it to `OrderService`.

```
Spring Container
      |
      +----> PaymentService
      |
      +----> OrderService
                  |
                  | injected dependency
                  v
            PaymentService
```

---

# 29\. IoC vs DI — Interview Question

### Question

> What is the difference between IoC and DI?

### Answer

**IoC** is the broader principle where control over object creation/dependency management is transferred from application code to a container/framework.

**DI** is a technique used to implement IoC by supplying an object's dependencies from outside instead of having the object create them itself.

### Easy Memory Trick

```
IoC = Principle
DI  = Technique
```

---

# 30\. Spring Ecosystem

Spring is not just one module.

A simplified view:

```
                 +----------------------+
                 |     SPRING BOOT      |
                 |  Automation/Defaults |
                 +----------+-----------+
                            |
        +-------------------+-------------------+
        |                   |                   |
        v                   v                   v
  Spring MVC          Spring Data        Spring Security
        |                   |                   |
        +-------------------+-------------------+
                            |
                            v
                     Spring Framework
                            |
                            v
                      Spring Core
                            |
                     IoC / DI / Beans
```

---

# 31\. Spring Core

Spring Core provides the foundation for Spring's container model.

Important concepts:

```
IoC
DI
Beans
ApplicationContext
Bean lifecycle
Configuration
```

Think:

```
Spring Core
     |
     +-- IoC
     +-- DI
     +-- Bean management
     +-- Container
```

---

# 32\. Spring MVC

Spring MVC is used for web applications and HTTP APIs.

It provides mechanisms for:

- Request mapping
- Controllers
- Request handling
- Data binding
- Validation
- Response handling

Example:

```
@RestController
class CourseController {

    @GetMapping("/courses")
    public List<Course> getCourses() {
        return courseService.getCourses();
    }
}
```

Instead of manually writing:

```
if (endpoint.equals("/courses")) {
    ...
}
```

we declare:

```
@GetMapping("/courses")
```

Spring handles the mapping.

---

# 33\. Does Spring MVC Replace Servlets?

**No.**

This is a very important interview question.

Spring MVC is built on top of the Servlet-based web infrastructure in traditional servlet deployments.

A simplified request flow is:

```
Client
  |
  | HTTP Request
  v
Tomcat / Servlet Container
  |
  v
DispatcherServlet
  |
  v
Controller
  |
  v
Service
  |
  v
Repository
  |
  v
Database
```

The key component is:

> **DispatcherServlet**

It acts as a front controller for Spring MVC requests.

---

# 34\. DispatcherServlet

The `DispatcherServlet` receives incoming requests and helps route them to the appropriate controller.

Example:

```
GET /courses
      |
      v
DispatcherServlet
      |
      | finds matching handler
      v
CourseController
      |
      v
getCourses()
```

This removes much of the manual routing code developers would otherwise need to write.

---

# 35\. Spring Data

Spring Data provides abstractions that simplify data access.

Instead of writing a lot of repetitive persistence code, developers can use repository abstractions.

Example:

```
public interface CourseRepository
        extends JpaRepository<Course, Long> {
}
```

Then:

```
courseRepository.findAll();
```

The developer doesn't need to manually implement basic CRUD operations.

---

# 36\. Spring Security

Spring Security provides security features such as:

```
Authentication
Authorization
Password handling
Security filters
Session/security context
OAuth2/resource server integrations
```

### Authentication vs Authorization

```
Authentication
     ↓
Who are you?

Authorization
     ↓
What are you allowed to do?
```

Example:

```
User logs in
    ↓
Authentication
    ↓
User identified as ADMIN
    ↓
Authorization
    ↓
Can access /admin
```

---

# 37\. Spring AOP

AOP = Aspect-Oriented Programming.

It is useful for **cross-cutting concerns**.

Examples:

```
Logging
Transactions
Security-related concerns
Auditing
Metrics
```

Instead of mixing logging into every business method:

```
public void createOrder() {

    log("starting");

    // business logic

    log("finished");
}
```

AOP can allow such behavior to be applied separately.

Conceptually:

```
             Business Logic
                  |
        +---------+---------+
        |                   |
     Logging            Transaction
        |                   |
        +---------+---------+
                  |
               Method
```

---

# 38\. Spring Boot

Spring Boot is built on the Spring ecosystem and makes application setup much easier.

It provides:

- Auto-configuration
- Starter dependencies
- Embedded server support
- Sensible defaults
- Externalized configuration
- Production-oriented features through the broader Boot ecosystem

---

# 39\. Does Spring Boot Replace Spring?

**No.**

A good interview answer:

> Spring Boot is a project in the Spring ecosystem that simplifies the configuration and bootstrapping of Spring applications. It does not replace Spring Framework; it builds on Spring and provides conventions, auto-configuration, starters, and other conveniences.

---

# 40\. Spring vs Spring Boot

| Spring Framework | Spring Boot |
| --- | --- |
| Core framework/ecosystem | Simplifies Spring application setup |
| More configuration may be required | Convention and auto-configuration |
| Provides modules such as Core, MVC, etc. | Helps assemble/run Spring applications |
| Foundation | Automation/convenience layer |

Memory trick:

```
Spring
   ↓
Framework

Spring Boot
   ↓
Easy way to build/run Spring applications
```

---

# 41\. Spring Boot Embedded Server

A major convenience is that a Spring Boot web application can run with an embedded servlet container.

Conceptually:

```
Traditional approach:

Java Application
      +
External Tomcat
```

Spring Boot commonly gives:

```
+--------------------------------+
|       Spring Boot Application  |
|                                |
|  Application Code              |
|  Spring Framework              |
|  Embedded Servlet Container    |
+--------------------------------+
```

So you can commonly run:

```
java -jar application.jar
```

instead of manually deploying a WAR to an external server.

---

# 42\. Database Access Evolution

Java database access evolved through several layers.

A simplified conceptual stack:

```
+----------------------+
|   Spring Data JPA    |
+----------------------+
           |
           v
+----------------------+
|         JPA          |
|    Specification     |
+----------------------+
           |
           v
+----------------------+
|      Hibernate       |
|    JPA Provider      |
+----------------------+
           |
           v
+----------------------+
|         JDBC         |
+----------------------+
           |
           v
+----------------------+
|      Database        |
+----------------------+
```

---

# 43\. JDBC

JDBC = **Java Database Connectivity**

It is a standard Java API for interacting with relational databases.

Example:

```
Connection connection =
        DriverManager.getConnection(url, username, password);

PreparedStatement statement =
        connection.prepareStatement(
            "SELECT * FROM courses"
        );

ResultSet resultSet =
        statement.executeQuery();
```

You have to deal with things such as:

```
Connection
PreparedStatement
ResultSet
SQL
Exception handling
Resource management
Mapping results to objects
```

---

# 44\. Why JDBC Can Become Repetitive

Suppose the database contains:

```
courses
-------------------
id
name
price
```

Java contains:

```
class Course {
    Long id;
    String name;
    BigDecimal price;
}
```

With JDBC, you may need to manually map:

```
Course course = new Course();

course.setId(resultSet.getLong("id"));
course.setName(resultSet.getString("name"));
course.setPrice(resultSet.getBigDecimal("price"));
```

This repetitive mapping is one of the problems ORM tools help address.

---

# 45\. JPA

JPA = **Jakarta Persistence** in modern terminology.

Historically it was called Java Persistence API.

The important interview point is:

> JPA is a specification, not an implementation.

Think:

```
JPA
 ↓
Rules / Contracts / Specification
```

It defines concepts for ORM and persistence.

---

# 46\. ORM

ORM = Object-Relational Mapping.

It maps:

```
Java Object
     ↕
Database Row
```

Example:

```
Java:

Course
----------------
id
name
price

        ↕

Database:

courses
----------------
id
name
price
```

An ORM allows developers to work more naturally with objects while the persistence provider handles much of the SQL/database interaction.

---

# 47\. Hibernate

Hibernate is a popular ORM framework and a JPA provider.

Think:

```
JPA = Specification
Hibernate = Implementation/provider
```

For example:

```
@Entity
class Course {

    @Id
    private Long id;

    private String name;

    private BigDecimal price;
}
```

Hibernate can manage persistence for this entity.

---

# 48\. JPA vs Hibernate — Interview Question

### Question

> Is JPA a framework?

Best answer:

> JPA is a persistence specification/API, not a concrete implementation. Hibernate is one of the implementations/providers that implements the JPA specification.

Memory:

```
JPA       = Rule book
Hibernate = Implements the rule book
```

---

# 49\. Spring Data JPA

Spring Data JPA sits above JPA and makes repository-based data access easier.

Example:

```
public interface CourseRepository
        extends JpaRepository<Course, Long> {

}
```

You automatically get operations such as:

```
findAll()
findById()
save()
delete()
count()
existsById()
```

You can also define derived queries such as:

```
List<Course> findByName(String name);
```

Spring Data can derive the query from the method name.

---

# 50\. Complete Database Stack

A useful interview diagram:

```
Your Application
       |
       v
Spring Data JPA
       |
       v
JPA API
       |
       v
Hibernate
       |
       v
JDBC
       |
       v
Database
```

### Important Nuance

Do not interpret this as:

> Every database call literally travels through these layers in exactly this simplistic sequence.

It is a conceptual stack showing the responsibilities/abstractions.

---

# 51\. Complete Spring Boot Request Flow

This is one of the most important diagrams to remember.

Suppose the client requests:

```
GET /courses/10
```

Flow:

```
                         HTTP
                          |
                          v
+----------+       +-------------+
|  Client  | ----> |    Tomcat   |
+----------+       |  Container  |
                   +------+------+
                          |
                          v
                  +---------------+
                  |DispatcherServlet|
                  +-------+-------+
                          |
                          v
                  +---------------+
                  |  Controller   |
                  +-------+-------+
                          |
                          v
                  +---------------+
                  |    Service    |
                  +-------+-------+
                          |
                          v
                  +---------------+
                  |  Repository   |
                  +-------+-------+
                          |
                          v
                  +---------------+
                  |   Hibernate   |
                  +-------+-------+
                          |
                          v
                  +---------------+
                  |     JDBC      |
                  +-------+-------+
                          |
                          v
                  +---------------+
                  |   Database    |
                  +---------------+
```

Response travels back upward:

```
Database
   ↓
JDBC
   ↓
Hibernate
   ↓
Repository
   ↓
Service
   ↓
Controller
   ↓
DispatcherServlet
   ↓
Tomcat
   ↓
Client
```

---

# 52\. Typical Spring Boot Layered Architecture

A common application structure is:

```
                CLIENT
                  |
                  v
             Controller
                  |
                  v
               Service
                  |
                  v
             Repository
                  |
                  v
              Database
```

## Controller

Responsible primarily for HTTP/API concerns.

Example:

```
@RestController
@RequestMapping("/courses")
class CourseController {

    @GetMapping("/{id}")
    public Course getCourse(@PathVariable Long id) {
        return courseService.getCourse(id);
    }
}
```

---

## Service

Contains business logic.

```
@Service
class CourseService {

    public Course getCourse(Long id) {
        // business rules
        return repository.findById(id).orElseThrow();
    }
}
```

---

## Repository

Responsible for persistence/data access.

```
@Repository
interface CourseRepository
        extends JpaRepository<Course, Long> {
}
```

---

# 53\. Monolithic Architecture

A monolith is an application deployed as a single unit.

Example:

```
+-----------------------------------------+
|           E-COMMERCE APPLICATION        |
|                                         |
|  User Module                            |
|  Order Module                           |
|  Payment Module                         |
|  Product Module                         |
|  Notification Module                   |
|                                         |
+-----------------------------------------+
                    |
                    v
                Database
```

The modules may be logically separated in code, but they are commonly deployed together as one application unit.

---

# 54\. Advantages of a Monolith

- Simple to develop initially
- Simple deployment
- Easier local development
- Easier debugging initially
- Often simpler communication between modules
- Can be efficient for smaller systems

---

# 55\. Disadvantages of a Monolith

As the application grows:

```
Codebase
   ↓
Very large
   ↓
Harder to understand
   ↓
Harder to deploy independently
   ↓
Scaling individual components becomes difficult
```

For example:

```
Payment traffic increases

But you need to deploy:

User
+
Order
+
Product
+
Payment
```

even if only Payment requires more resources.

---

# 56\. Microservices Architecture

Microservices architecture breaks an application into independently deployable services.

Example:

```
                    +----------------+
                    | User Service   |
                    +----------------+
                            ^
                            |
                            |
+---------+          +------+-------+          +----------------+
| Client  | -------> | Order Service| -------> |Payment Service|
+---------+          +------+-------+          +----------------+
                            |
                            v
                    +----------------+
                    | Product Service|
                    +----------------+
```

Each service focuses on a business capability.

---

# 57\. Example E-Commerce Microservices

Possible services:

```
User Service
Order Service
Payment Service
Product Service
Inventory Service
Notification Service
Shipping Service
```

Each service can potentially be:

- Developed independently
- Deployed independently
- Scaled independently
- Owned by a separate team

---

# 58\. Communication Between Microservices

Services often communicate using HTTP/REST APIs.

Example:

```
Order Service
     |
     | POST /payments
     v
Payment Service
     |
     | response
     v
Order Service
```

But microservices can also communicate through asynchronous messaging systems, depending on the architecture.

---

# 59\. Monolith vs Microservices

| Feature | Monolith | Microservices |
| --- | --- | --- |
| Deployment | Usually one unit | Multiple independent units |
| Codebase | Usually one application | Multiple services |
| Scaling | Often application-wide | Can scale individual services |
| Complexity | Lower initially | Higher operational complexity |
| Deployment | Simpler | More complex |
| Communication | In-process calls | Network calls often involved |
| Failure isolation | Lower | Potentially better |
| Technology choices | Usually more uniform | Can vary between services |
| Infrastructure | Simpler | More infrastructure/observability needed |

---

# 60\. Microservices Are NOT Automatically Better

This is an important interview point.

Do not say:

> Microservices are always better than monoliths.

Instead:

> Microservices solve certain organizational and scaling problems, but they introduce distributed-system complexity.

Microservices can introduce:

```
Network failures
Latency
Service discovery
Distributed tracing
Centralized logging
Data consistency problems
Deployment complexity
Monitoring complexity
Authentication between services
Retries/timeouts
Circuit breakers
```

---

# 61\. The Evolution — Big Picture

The entire lecture can be remembered as an evolution:

```
RAW NETWORKING
     |
     v
java.net
     |
     | Too much low-level work
     v
SERVLETS
     |
     | Networking/web infrastructure simplified
     v
SPRING
     |
     | Loose coupling / DI / IoC
     v
SPRING MVC
     |
     | Easier web/API development
     v
SPRING DATA
     |
     | Easier persistence
     v
SPRING SECURITY
     |
     | Authentication/Authorization
     v
SPRING BOOT
     |
     | Automation + conventions
     v
MODERN SPRING APPLICATIONS
```

---

# 62\. One Request — End-to-End Story

Suppose a mobile application wants course #10.

It sends:

```
GET /courses/10
Accept: application/json
```

### Step 1 — Client

Mobile app creates the HTTP request.

```
Mobile App
    |
    | GET /courses/10
    v
```

### Step 2 — Network/Server

Request reaches the server's network endpoint.

```
IP Address + Port
        |
        v
Tomcat
```

### Step 3 — Servlet Container

Tomcat processes the HTTP request and provides the Servlet infrastructure.

```
Tomcat
   |
   v
DispatcherServlet
```

### Step 4 — Spring MVC

DispatcherServlet finds the appropriate controller.

```
DispatcherServlet
       |
       v
CourseController
```

### Step 5 — Service Layer

Controller delegates business logic.

```
CourseController
       |
       v
CourseService
```

### Step 6 — Repository

Service asks the repository for data.

```
CourseService
       |
       v
CourseRepository
```

### Step 7 — Persistence

The persistence layer uses JPA/Hibernate and ultimately JDBC/database connectivity.

```
Repository
   |
   v
Hibernate
   |
   v
JDBC
   |
   v
Database
```

### Step 8 — Response

The result travels back:

```
Database
   ↓
Repository
   ↓
Service
   ↓
Controller
   ↓
DispatcherServlet
   ↓
Tomcat
   ↓
Mobile App
```

Example:

```
{
  "id": 10,
  "name": "Spring Boot",
  "price": 999
}
```

---

# 63\. Extremely Important Interview Concepts

Memorize these distinctions.

## HTTP vs HTTPS

```
HTTP   = application protocol
HTTPS  = HTTP over TLS-secured communication
```

---

## IP vs Port

```
IP   = identifies network host/address
Port = identifies a network service endpoint on that host
```

---

## Servlet vs Servlet Container

```
Servlet
   ↓
Java component handling web requests

Servlet Container
   ↓
Runtime that manages servlet execution and web infrastructure
```

---

## JPA vs Hibernate

```
JPA
 ↓
Specification/API

Hibernate
 ↓
JPA provider/implementation
```

---

## Spring vs Spring Boot

```
Spring
 ↓
Framework/ecosystem

Spring Boot
 ↓
Simplifies configuration and bootstrapping of Spring applications
```

---

## IoC vs DI

```
IoC
 ↓
Principle

DI
 ↓
Technique for supplying dependencies
```

---

## Authentication vs Authorization

```
Authentication
 ↓
Who are you?

Authorization
 ↓
What are you allowed to do?
```

---

## PUT vs PATCH

```
PUT
 ↓
Generally complete replacement/update

PATCH
 ↓
Partial modification
```

---

## 401 vs 403

```
401
 ↓
Authentication problem

403
 ↓
Authenticated/known identity but insufficient permission
```

---

# 64\. Interview Questions — Beginner Level

## Q1. What is client-server architecture?

**Answer:**

It is an architecture where a client initiates a request and a server processes that request and returns a response.

---

## Q2. Is a browser the only type of client?

**Answer:**

No. A client can be a browser, mobile application, frontend application, Postman, or another server/service.

---

## Q3. What is HTTP?

**Answer:**

HTTP is an application-layer protocol used for communication between clients and servers.

---

## Q4. What are common HTTP methods?

**Answer:**

GET, POST, PUT, PATCH, and DELETE.

---

## Q5. What is the difference between GET and POST?

**Answer:**

GET is generally used to retrieve resources, while POST is commonly used to submit data or create a resource.

---

## Q6. What is a port?

**Answer:**

A port identifies a network service endpoint on a host. It allows multiple network services to operate on the same IP address.

---

## Q7. What is `127.0.0.1`?

**Answer:**

It is the IPv4 loopback address, referring to the local host.

---

## Q8. What is a Servlet?

**Answer:**

A Servlet is a Java component designed to handle web requests and generate responses within a servlet container.

---

## Q9. What is Tomcat?

**Answer:**

Apache Tomcat is a Servlet container that provides a runtime environment for servlet-based Java web applications.

---

## Q10. What problem do Servlets solve?

**Answer:**

They abstract much of the low-level networking and HTTP request/response handling that developers would otherwise have to implement using raw sockets.

---

# 65\. Interview Questions — Spring

## Q11. What problem does Spring solve?

**Answer:**

Spring helps build maintainable Java applications by providing features such as dependency injection, IoC, configuration, web development support, data access, security, and more.

---

## Q12. What is Dependency Injection?

**Answer:**

Dependency Injection is a technique where an object's dependencies are provided to it from outside rather than the object creating those dependencies itself.

---

## Q13. What is IoC?

**Answer:**

Inversion of Control means transferring control of object creation and dependency management from application code to a framework/container.

---

## Q14. What is a Spring Bean?

**Answer:**

A Spring Bean is an object that is instantiated, configured, and managed by the Spring IoC container.

---

## Q15. What is Spring MVC?

**Answer:**

Spring MVC is a Spring web framework used to build web applications and HTTP APIs. It provides mechanisms such as controllers and request mappings and is built around the Servlet-based web model.

---

## Q16. What is DispatcherServlet?

**Answer:**

DispatcherServlet is the central front-controller component in Spring MVC. It receives requests and coordinates finding the appropriate handler/controller and processing the response.

---

## Q17. Does Spring MVC use Servlets?

**Answer:**

Yes. Traditional Spring MVC is built on the Servlet API. DispatcherServlet itself is a Servlet.

---

## Q18. Does Spring Boot replace Spring?

**Answer:**

No. Spring Boot builds on the Spring ecosystem and simplifies application configuration, dependency setup, and bootstrapping.

---

# 66\. Interview Questions — Database

## Q19. What is JDBC?

**Answer:**

JDBC is the standard Java API for interacting with relational databases.

---

## Q20. What is JPA?

**Answer:**

JPA is a specification/API for persistence and ORM in Java/Jakarta applications. It defines interfaces and rules rather than being a concrete ORM implementation.

---

## Q21. What is Hibernate?

**Answer:**

Hibernate is an ORM framework and a popular JPA provider that implements the JPA specification.

---

## Q22. What is Spring Data JPA?

**Answer:**

Spring Data JPA is part of Spring Data and provides repository abstractions that simplify working with JPA-based persistence.

---

## Q23. Is Hibernate the same as JPA?

**Answer:**

No.

```
JPA       = Specification
Hibernate = Implementation/provider
```

---

## Q24. What is ORM?

**Answer:**

ORM stands for Object-Relational Mapping. It maps application objects/entities to relational database structures.

---

# 67\. Interview Questions — Architecture

## Q25. What is a monolithic application?

**Answer:**

A monolithic application is generally deployed as a single application unit containing multiple business capabilities/modules.

---

## Q26. What are microservices?

**Answer:**

Microservices architecture structures an application as multiple independently deployable services, each typically focused on a specific business capability.

---

## Q27. Why use microservices?

**Answer:**

Potential benefits include independent deployment, independent scaling, team autonomy, and clearer service boundaries.

---

## Q28. What are disadvantages of microservices?

**Answer:**

They introduce distributed-system complexity such as network failures, latency, service discovery, observability, deployment complexity, and distributed data consistency concerns.

---

## Q29. Can one microservice call another?

**Answer:**

Yes. They can communicate through mechanisms such as HTTP APIs or asynchronous messaging.

---

## Q30. Is microservices architecture always better than monolithic architecture?

**Answer:**

No. A monolith can be simpler and more appropriate for many systems, especially when the system is small or the team is small. Microservices should be adopted when their benefits justify their operational and architectural complexity.

---

# 68\. Scenario-Based Interview Questions

These are more important for experienced interviews.

## Scenario 1

> Your Spring Boot API is receiving a request. Explain the request flow.

A strong answer:

```
Client
 ↓
Network
 ↓
Embedded Servlet Container (e.g. Tomcat)
 ↓
DispatcherServlet
 ↓
Controller
 ↓
Service
 ↓
Repository
 ↓
JPA/Hibernate
 ↓
JDBC
 ↓
Database
```

Then the response follows the reverse path.

---

## Scenario 2

> Why don't we write raw ServerSocket code in Spring Boot?

Because Spring Boot applications typically rely on established web infrastructure and an embedded web server/container that handles low-level networking, HTTP processing, request dispatching, concurrency, and other infrastructure concerns.

---

## Scenario 3

> Why is Dependency Injection useful?

It reduces coupling and makes components easier to:

- Replace
- Test
- Maintain
- Configure

For example:

```
OrderService
     |
     v
PaymentService
```

can depend on an interface:

```
interface PaymentGateway {
    void pay();
}
```

and different implementations can be injected:

```
PaymentGateway
    |
    +---- StripePaymentGateway
    |
    +---- MockPaymentGateway
```

This is especially useful in testing.

---

## Scenario 4

> Your application has 100 modules and every class creates its dependencies using `new`. What problem might you face?

Potentially severe coupling.

Example:

```
class OrderService {
    private PaymentService payment =
        new PaymentService();
}
```

If `PaymentService` changes, many classes may need modification.

Dependency injection centralizes dependency creation/configuration.

---

# 69\. Common Interview Traps

## Trap 1

❌ "JPA is an ORM framework."

Better:

✅ "JPA is a persistence specification/API; Hibernate is a popular implementation/provider."

---

## Trap 2

❌ "Spring Boot is a replacement for Spring."

Better:

✅ "Spring Boot simplifies the setup and bootstrapping of Spring applications."

---

## Trap 3

❌ "Spring MVC doesn't use Servlets."

Better:

✅ "Traditional Spring MVC is built on the Servlet API, with DispatcherServlet acting as the front controller."

---

## Trap 4

❌ "A client is always a browser."

Better:

✅ "A client is anything that initiates a request."

---

## Trap 5

❌ "HTTPS means the server is completely secure."

Better:

✅ "HTTPS protects HTTP communication in transit using TLS; it does not guarantee that the application itself has no vulnerabilities."

---

## Trap 6

❌ "Microservices are always better."

Better:

✅ "Microservices provide independent deployment/scaling and service boundaries, but introduce distributed-system complexity."

---

## Trap 7

❌ "PUT always means update one field."

Better:

✅ "PUT is generally used for complete replacement/update semantics, while PATCH is designed for partial modification."

---

# 70\. Quick Revision Sheet

```
CLIENT
  ↓
Sends request

HTTP
  ↓
Communication protocol

SERVER
  ↓
Processes request

IP
  ↓
Identifies host/network endpoint

PORT
  ↓
Identifies service endpoint on host

SERVLET
  ↓
Java web component

TOMCAT
  ↓
Servlet container

SPRING CORE
  ↓
IoC + DI + Beans

SPRING MVC
  ↓
Web/API layer

DISPATCHERSERVLET
  ↓
Front controller

SPRING DATA JPA
  ↓
Repository/data-access abstraction

JPA
  ↓
Persistence specification

HIBERNATE
  ↓
JPA provider / ORM

JDBC
  ↓
Java database connectivity

DATABASE
  ↓
Persistent data

SPRING BOOT
  ↓
Automation + conventions + easy application bootstrapping

MONOLITH
  ↓
One deployable application unit

MICROSERVICES
  ↓
Multiple independently deployable services
```

---

# 71\. Most Important Diagram to Memorize

```
                         CLIENT
                           |
                           | HTTP Request
                           v
                 +--------------------+
                 | Embedded Web Server|
                 | / Servlet Container|
                 +---------+----------+
                           |
                           v
                 +--------------------+
                 | DispatcherServlet  |
                 +---------+----------+
                           |
                           v
                 +--------------------+
                 |    Controller      |
                 +---------+----------+
                           |
                           v
                 +--------------------+
                 |     Service        |
                 +---------+----------+
                           |
                           v
                 +--------------------+
                 |    Repository      |
                 +---------+----------+
                           |
                           v
                 +--------------------+
                 |   Spring Data JPA  |
                 +---------+----------+
                           |
                           v
                 +--------------------+
                 |       JPA          |
                 +---------+----------+
                           |
                           v
                 +--------------------+
                 |     Hibernate      |
                 +---------+----------+
                           |
                           v
                 +--------------------+
                 |       JDBC         |
                 +---------+----------+
                           |
                           v
                 +--------------------+
                 |      DATABASE      |
                 +--------------------+
```

---

# 72\. Final Mental Model

When learning Spring, don't memorize Spring annotations blindly.

Understand the **layers**:

```
                    USER
                     |
                     v
                  CLIENT
                     |
                  HTTP
                     |
                     v
             WEB SERVER / TOMCAT
                     |
                     v
              SPRING MVC
                     |
                Controller
                     |
                  Service
                     |
               Repository
                     |
               Spring Data
                     |
                    JPA
                     |
                 Hibernate
                     |
                   JDBC
                     |
                 DATABASE
```

And understand **why each layer exists**:

```
Raw Sockets
    ↓
Too much low-level networking

Servlets
    ↓
Simplify Java web development

Spring
    ↓
Reduce coupling / manage dependencies

Spring MVC
    ↓
Simplify HTTP/web development

Spring Data JPA
    ↓
Simplify persistence

Spring Security
    ↓
Security

Spring Boot
    ↓
Simplify configuration and application bootstrapping

Microservices
    ↓
Independently deployable business services
```

---

# 73\. Interview Preparation Checklist

Before moving to the next lecture, you should be able to answer these **without notes**:

### Networking

- [ ] What is client-server architecture?
- [ ] What is HTTP?
- [ ] What is HTTPS?
- [ ] Difference between HTTP and HTTPS?
- [ ] What is an HTTP request?
- [ ] What is an HTTP response?
- [ ] What are HTTP headers?
- [ ] What is Content-Type?
- [ ] What is Accept?
- [ ] GET vs POST?
- [ ] PUT vs PATCH?
- [ ] What does 200 mean?
- [ ] 201 vs 204?
- [ ] 401 vs 403?
- [ ] 404 vs 500?
- [ ] What is an IP address?
- [ ] What is a port?
- [ ] What is `127.0.0.1`?

### Java Web Development

- [ ] Difference between normal Java application and server application?
- [ ] What is `ServerSocket`?
- [ ] Why is raw socket programming difficult for HTTP?
- [ ] What is a Servlet?
- [ ] What is a Servlet Container?
- [ ] What is Tomcat?
- [ ] What are `HttpServletRequest` and `HttpServletResponse`?
- [ ] What is the Servlet lifecycle?
- [ ] How does a Servlet handle GET/POST?

### Spring

- [ ] What is Spring?
- [ ] What is IoC?
- [ ] What is DI?
- [ ] IoC vs DI?
- [ ] What is a Spring Bean?
- [ ] What is Spring Core?
- [ ] What is Spring MVC?
- [ ] What is DispatcherServlet?
- [ ] Does Spring MVC use Servlets?
- [ ] What is Spring Data?
- [ ] What is Spring Security?
- [ ] What is AOP?
- [ ] What is Spring Boot?
- [ ] Spring vs Spring Boot?

### Database

- [ ] What is JDBC?
- [ ] What is ORM?
- [ ] What is JPA?
- [ ] Is JPA an implementation?
- [ ] What is Hibernate?
- [ ] JPA vs Hibernate?
- [ ] What is Spring Data JPA?
- [ ] Explain the database access stack.

### Architecture

- [ ] What is a monolith?
- [ ] What are microservices?
- [ ] Monolith vs microservices?
- [ ] Advantages of microservices?
- [ ] Disadvantages of microservices?
- [ ] How do microservices communicate?
- [ ] Why shouldn't every application immediately become microservices?

---

# 74\. One-Line Interview Answers

For rapid revision:

| Question | One-line answer |
| --- | --- |
| HTTP? | Protocol for client-server web communication |
| HTTPS? | HTTP secured with TLS |
| IP? | Identifies a network host/address |
| Port? | Identifies a service endpoint on a host |
| Servlet? | Java web component that handles requests |
| Tomcat? | Servlet container |
| IoC? | Control of object management is transferred to a container/framework |
| DI? | Dependencies are supplied from outside an object |
| Bean? | Object managed by Spring's IoC container |
| Spring MVC? | Spring's Servlet-based web framework |
| DispatcherServlet? | Spring MVC's central front controller |
| JDBC? | Java API for relational database connectivity |
| JPA? | Persistence specification/API |
| Hibernate? | ORM framework and JPA provider |
| Spring Data JPA? | Simplifies JPA repository/data access |
| Spring Boot? | Simplifies Spring application setup and bootstrapping |
| Monolith? | Application deployed as one unit |
| Microservices? | Independently deployable services organized around business capabilities |

---

# 75\. Golden Rule

Whenever you learn a new Spring feature, ask yourself:

> **"What problem did this layer solve?"**

For example:

```
Raw Socket
    ↓
Problem: Networking is hard
    ↓
Servlet
    ↓
Problem: Web request handling is repetitive
    ↓
Spring MVC
    ↓
Problem: Application configuration/coupling
    ↓
Spring Core + DI
    ↓
Problem: Database boilerplate
    ↓
JPA/Hibernate
    ↓
Problem: Repository boilerplate
    ↓
Spring Data JPA
    ↓
Problem: Spring configuration/bootstrapping
    ↓
Spring Boot
```
