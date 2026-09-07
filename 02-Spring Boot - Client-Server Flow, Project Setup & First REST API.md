# Spring Boot - Client-Server Flow, Project Setup & First REST API

> **Purpose:** Understand what actually happens when a browser calls a Spring Boot application, how local and production environments differ, how Spring Boot projects are created, how Tomcat works, and how the first REST endpoint is built.

---

# 1\. Big Picture

Before writing Spring Boot code, understand this complete journey:

```
                         INTERNET
                            |
                            v
                     +-------------+
                     |     DNS     |
                     | Domain -> IP|
                     +------+------+
                            |
                            v
                     +-------------+
                     |    SERVER   |
                     | IP + Port   |
                     +------+------+
                            |
                            v
                  +---------------------+
                  | Reverse Proxy       |
                  | Nginx / Load Balancer|
                  +----------+----------+
                             |
                             v
                  +---------------------+
                  | Spring Boot App     |
                  | Embedded Tomcat      |
                  +----------+----------+
                             |
                             v
                       Controller
                             |
                             v
                        Business Logic
```

For local development, much of the internet infrastructure disappears:

```
Browser
   |
   | HTTP
   v
localhost:8080
   |
   v
Spring Boot
   |
   v
Embedded Tomcat
   |
   v
Controller
```

---

# 2\. Localhost — What Does It Mean?

When developing locally, your computer can act as both:

```
Client + Server
```

Example:

```
http://localhost:8080/hello
```

Here:

```
localhost
    ↓
Your own computer

8080
    ↓
Port where your application is listening

/hello
    ↓
Endpoint/path
```

---

# 3\. What Is `127.0.0.1`?

`127.0.0.1` is the IPv4 loopback address.

It means:

> Send the network traffic back to this same machine.

Therefore:

```
localhost
    ≈
127.0.0.1
```

Example:

```
http://127.0.0.1:8080/hello
```

and:

```
http://localhost:8080/hello
```

normally reach the same local machine.

---

# 4\. Local Client-Server Architecture

Suppose Spring Boot is running on your computer.

```
+----------------------+          HTTP Request
|                      | ---------------------------->
|   Google Chrome      |                             |
|      CLIENT          |                             |
+----------------------+                             |
                                                     v
                                         +----------------------+
                                         |                      |
                                         |   Spring Boot App    |
                                         |       SERVER         |
                                         |                      |
                                         +----------------------+
                                                     |
                                                     |
                                         HTTP Response
<----------------------------------------------------+
```

The important point:

> The client and server can exist on the **same physical computer**.

---

# 5\. What Happens When You Visit `localhost:8080`?

Suppose your Spring Boot application is running on port 8080.

You enter:

```
http://localhost:8080/hello
```

Conceptually:

```
Browser
   |
   | 1. Resolve localhost
   v
127.0.0.1
   |
   | 2. Connect to port 8080
   v
Tomcat
   |
   | 3. Find /hello
   v
Spring Controller
   |
   | 4. Execute Java method
   v
Response
   |
   v
Browser
```

---

# 6\. Real-World Production Flow

Now suppose the user visits:

```
https://www.example.com
```

The browser cannot use the domain name directly as the destination for network routing.

It needs to resolve the domain to an IP address.

This is where **DNS** comes in.

---

# 7\. DNS — Domain Name System

DNS stands for:

> **Domain Name System**

Its main purpose is to translate domain names into IP addresses.

Conceptually:

```
www.example.com
       |
       | DNS lookup
       v
   203.0.113.10
```

The actual DNS system is more sophisticated than a simple single-server lookup, but this is the right conceptual model for beginners.

---

# 8\. Why Do We Need DNS?

Humans prefer:

```
www.example.com
```

Computers/network protocols ultimately communicate using IP addresses such as:

```
203.0.113.10
```

DNS provides the mapping.

Think:

```
Domain Name
     ↓
DNS
     ↓
IP Address
```

---

# 9\. Real-World Request Flow

Suppose you type:

```
https://www.example.com/courses
```

A simplified flow is:

```
+----------+
| Browser  |
+----+-----+
     |
     | 1. DNS lookup
     v
+----------+
|   DNS    |
+----+-----+
     |
     | 2. IP address
     v
+----------+
| Internet |
+----+-----+
     |
     | 3. HTTPS :443
     v
+----------------+
| Reverse Proxy  |
+-------+--------+
        |
        | 4. Forward internally
        v
+----------------+
| Spring Boot    |
| Application    |
+----------------+
```

---

# 10\. Does the Browser Really "Append 443"?

A useful conceptual model is:

```
HTTPS → port 443
HTTP  → port 80
```

If you write:

```
https://example.com
```

you normally don't need to explicitly write:

```
https://example.com:443
```

because `443` is the standard port associated with HTTPS.

Similarly:

```
http://example.com
```

normally implies port `80`.

---

# 11\. Standard Ports

| Protocol | Common Default Port |
| --- | --- |
| HTTP | 80 |
| HTTPS | 443 |
| Spring Boot default web port | 8080 |

Important:

> `8080` is a common default for Spring Boot's embedded web server, but it is not an HTTP-standard port.

---

# 12\. What Is a Port?

A port identifies a network service endpoint on a host.

Example:

```
192.168.1.10:8080
```

means:

```
192.168.1.10
       ↓
Host

8080
       ↓
Port
```

---

# 13\. IP Address vs Port

Remember:

```
IP Address
    ↓
Which machine/host?

Port
    ↓
Which network service on that host?
```

### Apartment Analogy

```
IP Address = Building Address
Port       = Apartment/Room Number
```

For example:

```
127.0.0.1:8080
```

Think:

```
Building = 127.0.0.1
Room     = 8080
```

This is only an analogy; ports identify network endpoints/services rather than literally identifying individual applications.

---

# 14\. One Computer, Many Ports

Your machine can have many services listening on different ports.

```
                 YOUR COMPUTER
              IP: 127.0.0.1
                    |
       +------------+------------+
       |            |            |
       v            v            v
    :3000         :8080        :5432
       |            |            |
    Frontend     Spring Boot   Database
```

The operating system uses the network connection's addressing information, including the destination port, to deliver traffic to the appropriate listening socket.

---

# 15\. What Is a Reverse Proxy?

A reverse proxy sits in front of backend servers.

```
Client
   |
   | HTTPS :443
   v
+----------------+
| Reverse Proxy  |
+-------+--------+
        |
        | Internal forwarding
        v
+----------------+
| Spring Boot    |
| :8080          |
+----------------+
```

Examples of technologies commonly used in this role include:

- Nginx
- Apache HTTP Server
- Cloud/load-balancing services
- Kubernetes ingress/gateway solutions

---

# 16\. Why Use a Reverse Proxy?

A reverse proxy can handle responsibilities such as:

- TLS termination
- Routing
- Load balancing
- Security controls
- Static content
- Compression
- Request filtering
- Forwarding requests to backend services

For example:

```
Internet
    |
    v
Reverse Proxy :443
    |
    +---------> Spring Boot :8080
    |
    +---------> Spring Boot :8081
    |
    +---------> Spring Boot :8082
```

This can distribute requests across multiple application instances.

---

# 17\. Important Interview Correction

A reverse proxy does not necessarily mean:

```
443 → 8080
```

That is only one possible configuration.

For example:

```
443 → 8080
443 → 8081
443 → 9000
```

or even:

```
443 → another machine
```

The important concept is:

> The public-facing endpoint and internal application endpoint can be different.

---

# 18\. Local vs Production

| Local Development | Production |
| --- | --- |
| Usually `localhost` | Public domain |
| Usually no public DNS needed | DNS is commonly involved |
| Browser may directly access app | Reverse proxy/load balancer often sits in front |
| Often port 8080 | Public HTTPS commonly uses 443 |
| Simple environment | Multiple infrastructure layers |

---

# 19\. Setting Up the Development Environment

Before creating a Spring Boot application, you generally need:

```
JDK
 +
IDE
 +
Build Tool
 +
Spring Boot Project
```

---

# 20\. JDK

JDK = **Java Development Kit**

It provides tools required to develop Java applications.

Important components include:

```
javac
Java compiler

java
Java launcher/runtime command

JVM
Executes Java bytecode
```

For modern Spring Boot development, use a Java version supported by the Spring Boot version you're using.

---

# 21\. Why Java Version Matters

Spring Boot versions support specific Java versions.

Therefore:

```
Spring Boot version
        +
Supported Java version
        ↓
Compatible development environment
```

### Interview Question

> Can I use any Java version with any Spring Boot version?

**Answer:**

No. Spring Boot versions have supported Java version ranges. You should check the compatibility requirements of the specific Spring Boot version you are using.

---

# 22\. IDE — IntelliJ IDEA

IDE = **Integrated Development Environment**

An IDE provides tools such as:

- Code editor
- Autocomplete
- Debugger
- Project navigation
- Refactoring
- Build integration
- Test execution
- Version-control integration

Common Java IDEs include:

- IntelliJ IDEA
- Eclipse
- NetBeans

---

# 23\. Spring Initializr

Spring Initializr is a project-generation service used to create Spring Boot project structures.

Website:

```
https://start.spring.io
```

Instead of manually creating:

```
pom.xml
src/main/java
src/main/resources
configuration
dependencies
```

Spring Initializr generates a ready-to-use project.

---

# 24\. Why Use Spring Initializr?

Without a project generator, you might need to manually configure:

```
Project structure
Build tool
Java version
Spring Boot version
Dependencies
Plugins
Configuration
```

Spring Initializr automates the initial setup.

---

# 25\. Spring Initializr Flow

```
Developer
    |
    v
Spring Initializr
    |
    | Select:
    | Maven
    | Java
    | Spring Boot version
    | Java version
    | Dependencies
    |
    v
Generated Project
    |
    v
Download ZIP
    |
    v
Open in IntelliJ
```

---

# 26\. Project Configuration

A typical Spring Boot project may contain:

| Setting | Example |
| --- | --- |
| Project | Maven |
| Language | Java |
| Packaging | Jar |
| Java | 21 |
| Dependency | Spring Web |

The exact Spring Boot version should be chosen based on your course/project requirements and current compatibility.

---

# 27\. Maven

Maven is a build and dependency management tool.

It helps with:

```
Dependency management
Compilation
Testing
Packaging
Plugins
Build lifecycle
```

---

# 28\. What Is a Dependency?

A dependency is an external library that your application requires.

For example:

```
Your Application
      |
      +---- Spring Web
      |
      +---- Spring Data JPA
      |
      +---- PostgreSQL Driver
      |
      +---- Spring Security
```

Without dependency management, you would have to manually download and manage JAR files.

---

# 29\. Maven's `pom.xml`

Maven configuration is primarily stored in:

```
pom.xml
```

POM means:

> **Project Object Model**

Example:

```
<dependencies>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

</dependencies>
```

Maven uses this information to resolve dependencies and manage the build.

---

# 30\. Maven Coordinates

A dependency is commonly identified using:

```
groupId
artifactId
version
```

Example:

```
<groupId>org.springframework.boot</groupId>
<artifactId>spring-boot-starter-web</artifactId>
```

Think:

```
groupId
   ↓
Organization/project group

artifactId
   ↓
Specific library
```

Spring Boot's dependency management can often manage compatible versions for Spring Boot dependencies, so you don't always need to specify every dependency version yourself.

---

# 31\. What Is a JAR?

JAR = **Java Archive**

It is a packaged archive used for Java applications/libraries.

Spring Boot commonly produces an executable JAR.

Example:

```
application.jar
```

You can commonly run it with:

```
java -jar application.jar
```

---

# 32\. JAR vs WAR

## JAR

Common modern Spring Boot packaging.

```
application.jar
```

Spring Boot can package the application with an embedded web server.

---

## WAR

WAR = **Web Application Archive**

Historically, Java web applications were often packaged as WAR files and deployed into an externally managed servlet container.

Modern Spring Boot applications commonly use executable JARs instead.

### Interview Question

> Does Spring Boot never support WAR?

**Answer:**

No. Spring Boot supports WAR packaging as well. JAR is simply the common and convenient choice for many modern Spring Boot applications.

---

# 33\. Spring Web Dependency

When you select:

```
Spring Web
```

you get the dependencies needed for building Spring web applications.

It brings in Spring MVC and the infrastructure needed to run a typical servlet-based web application.

In the common Spring Boot setup, an embedded servlet container such as Tomcat is included.

Therefore:

```
Spring Web
     |
     +---- Spring MVC
     |
     +---- Servlet web infrastructure
     |
     +---- Embedded Tomcat (typical setup)
```

---

# 34\. Embedded Tomcat

Traditionally:

```
Application
     |
     v
External Tomcat
```

With a typical Spring Boot executable JAR:

```
+--------------------------------+
|       Spring Boot JAR          |
|                                |
| Application Code               |
| Spring Framework               |
| Embedded Tomcat                |
+--------------------------------+
```

This means you can commonly start the application using:

```
java -jar application.jar
```

and the application starts its embedded web server.

---

# 35\. Project Structure

A typical Spring Boot Maven project:

```
demo/
│
├── src/
│   │
│   ├── main/
│   │   │
│   │   ├── java/
│   │   │   └── com/example/demo/
│   │   │       └── DemoApplication.java
│   │   │
│   │   └── resources/
│   │       └── application.properties
│   │
│   └── test/
│       └── java/
│
├── pom.xml
└── ...
```

---

# 36\. `src/main/java`

This is where your main application Java source code normally lives.

Example:

```
src/main/java/com/example/demo/
```

You may create:

```
controller/
service/
repository/
model/
config/
```

as your application grows.

Example:

```
src/main/java/com/example/demo/

├── DemoApplication.java
├── controller/
├── service/
├── repository/
└── model/
```

---

# 37\. `src/main/resources`

Contains application resources/configuration.

Common files include:

```
application.properties
application.yml
```

It may also contain:

```
static/
templates/
messages.properties
```

depending on the application.

---

# 38\. `application.properties`

This file contains external configuration.

Example:

```
server.port=9090
```

Other common configurations may include:

```
spring.datasource.url=...
spring.datasource.username=...
spring.datasource.password=...
```

This is preferable to hardcoding environment-specific configuration directly into Java code.

---

# 39\. `src/test`

Contains test source code.

Example:

```
src/test/java/
```

You may write:

```
Unit tests
Integration tests
Spring Boot tests
Controller tests
Repository tests
```

---

# 40\. `pom.xml`

The `pom.xml` is the Maven project configuration.

It can define:

```
Project metadata
Dependencies
Plugins
Build configuration
Spring Boot configuration
```

Think:

```
pom.xml
   ↓
How Maven should build/manage this project
```

---

# 41\. `DemoApplication.java`

The main application class commonly looks like:

```
package com.example.demo;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class DemoApplication {

    public static void main(String[] args) {
        SpringApplication.run(
            DemoApplication.class,
            args
        );
    }
}
```

---

# 42\. `main()` Method

Every Java application starts from a method such as:

```
public static void main(String[] args)
```

Spring Boot starts from this Java entry point.

But instead of manually creating every application object, we delegate startup to:

```
SpringApplication.run(...)
```

---

# 43\. `SpringApplication.run()`

This is the important startup call:

```
SpringApplication.run(
    DemoApplication.class,
    args
);
```

Conceptually, it:

```
Start Spring
     ↓
Create application context
     ↓
Process configuration
     ↓
Discover/register beans
     ↓
Auto-configure applicable infrastructure
     ↓
Start web server for a web application
     ↓
Application becomes ready
```

The exact startup internals are more complex, but this is the correct mental model.

---

# 44\. What Does `@SpringBootApplication` Mean?

This is one of the most important annotations.

```
@SpringBootApplication
```

It is a composed annotation that combines important Spring Boot annotations, notably:

```
@Configuration
@EnableAutoConfiguration
@ComponentScan
```

Conceptually:

```
@SpringBootApplication
        |
        +---- @Configuration
        |
        +---- @EnableAutoConfiguration
        |
        +---- @ComponentScan
```

---

# 45\. `@Configuration`

Indicates that the class can provide Spring configuration.

---

# 46\. `@EnableAutoConfiguration`

Tells Spring Boot to apply auto-configuration based on the application's dependencies and environment.

For example, if the application has web dependencies, Spring Boot can configure appropriate web infrastructure.

---

# 47\. `@ComponentScan`

Tells Spring to scan packages for Spring-managed components such as:

```
@Component
@Service
@Repository
@Controller
@RestController
```

A crucial practical rule:

> Put your main application class in a suitable parent package so component scanning can discover your application's components.

Example:

```
com.example.demo
│
├── DemoApplication.java
│
├── controller
│   └── HelloController.java
│
├── service
│   └── HelloService.java
│
└── repository
    └── HelloRepository.java
```

This works naturally because the application class is at the package root.

---

# 48\. What Happens When Spring Boot Starts?

A simplified startup sequence:

```
main()
  |
  v
SpringApplication.run()
  |
  v
Create Spring Application Context
  |
  v
Read configuration
  |
  v
Component scanning
  |
  v
Create/manage Beans
  |
  v
Auto-configuration
  |
  v
Start embedded web server
  |
  v
Application Ready
```

---

# 49\. Why Does `localhost:8080` Initially Show an Error?

Suppose you haven't created any controller.

You start the application.

Tomcat starts successfully:

```
Tomcat
  |
  v
Running on :8080
```

But you request:

```
GET /
```

Spring does not necessarily have a controller mapping for `/`.

Therefore you may see Spring Boot's **Whitelabel Error Page**.

This usually means:

> The server is running, but there is no handler/resource matching the requested URL.

---

# 50\. Whitelabel Error Page — Important Understanding

Do not think:

> Whitelabel Error Page = Tomcat is broken.

Usually the opposite is true.

It can indicate:

```
Application started
       ↓
Tomcat is running
       ↓
Request reached application
       ↓
No suitable mapping/resource
       ↓
Error response generated
```

---

# 51\. Creating the First Controller

Create:

```
HelloController.java
```

Example:

```
package com.example.demo;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HelloController {

    @GetMapping("/hello")
    public String hello() {
        return "Hello World";
    }
}
```

---

# 52\. What Does `@RestController` Mean?

```
@RestController
```

marks the class as a controller whose handler methods are intended to return response data directly.

It is effectively a convenience combination of:

```
@Controller
+
@ResponseBody
```

Conceptually:

```
@RestController
      |
      +---- Controller
      |
      +---- Return value → HTTP response body
```

---

# 53\. What Does `@GetMapping` Mean?

Example:

```
@GetMapping("/hello")
```

means:

> Map HTTP GET requests for `/hello` to this method.

Therefore:

```
GET /hello
      |
      v
hello()
```

---

# 54\. Method Name Does Not Determine the URL

This is important.

```
@GetMapping("/bye")
public String greetBy() {
    return "Bye";
}
```

The URL is:

```
/bye
```

not:

```
/greetBy
```

Why?

Because the mapping is explicitly defined by:

```
@GetMapping("/bye")
```

The Java method name can be different.

---

# 55\. First API — Complete Example

```
@RestController
public class HelloController {

    @GetMapping("/hello")
    public String hello() {
        return "Hello World";
    }

    @GetMapping("/bye")
    public String greetBy() {
        return "Bye";
    }
}
```

Requests:

```
GET http://localhost:8080/hello
```

Response:

```
Hello World
```

And:

```
GET http://localhost:8080/bye
```

Response:

```
Bye
```

---

# 56\. Why Does the Browser Display HTML?

Suppose you return:

```
return "<h1>Hello World</h1>";
```

The browser may render it as:

# Hello World

This is because the browser interprets the returned content as HTML when the response's media type is appropriate.

However, for a REST API, you will generally return structured data such as JSON rather than HTML markup.

---

# 57\. String vs JSON Response

String:

```
@GetMapping("/hello")
public String hello() {
    return "Hello World";
}
```

Response:

```
Hello World
```

Object:

```
@GetMapping("/course")
public Course getCourse() {
    return new Course(1L, "Spring Boot");
}
```

Spring MVC can serialize the object into JSON, typically using Jackson in a standard Spring Boot web setup:

```
{
  "id": 1,
  "name": "Spring Boot"
}
```

---

# 58\. Customizing the Server Port

Default:

```
8080
```

Change it in:

```
src/main/resources/application.properties
```

Add:

```
server.port=9090
```

Now:

```
Old:
http://localhost:8080/hello

New:
http://localhost:9090/hello
```

---

# 59\. What Happens When the Port Changes?

Before:

```
Browser
   |
   v
localhost:8080
   |
   v
Spring Boot
```

After:

```
Browser
   |
   v
localhost:9090
   |
   v
Spring Boot
```

If nothing is listening on `8080`, a request to:

```
localhost:8080
```

will fail at the connection level.

---

# 60\. Port Configuration Flow

```
application.properties
        |
        | server.port=9090
        v
Spring Boot
        |
        v
Embedded Tomcat
        |
        v
Listen on port 9090
```

---

# 61\. Complete First Application

### `DemoApplication.java`

```
package com.example.demo;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class DemoApplication {

    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }
}
```

### `HelloController.java`

```
package com.example.demo;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HelloController {

    @GetMapping("/hello")
    public String hello() {
        return "Hello World";
    }

    @GetMapping("/bye")
    public String bye() {
        return "Bye";
    }
}
```

### `application.properties`

```
server.port=8080
```

---

# 62\. Complete Request Flow for `/hello`

When you visit:

```
http://localhost:8080/hello
```

think:

```
                 BROWSER
                    |
                    | GET /hello
                    v
              localhost:8080
                    |
                    v
             Embedded Tomcat
                    |
                    v
            DispatcherServlet
                    |
                    v
             HelloController
                    |
                    v
                 hello()
                    |
                    v
              "Hello World"
                    |
                    v
               HTTP Response
                    |
                    v
                 BROWSER
```

This is one of the most important diagrams in the lecture.

---

# 63\. What Happens Behind `@GetMapping`?

When you write:

```
@GetMapping("/hello")
public String hello() {
    return "Hello World";
}
```

you are essentially declaring:

```
HTTP Method = GET
Path        = /hello
Handler     = hello()
```

Spring's web infrastructure maintains the mapping between requests and handler methods.

You don't manually write:

```
if (method.equals("GET")
        && path.equals("/hello")) {
    hello();
}
```

Spring handles the routing infrastructure.

---

# 64\. What Actually Calls `hello()`?

You don't write:

```
new HelloController().hello();
```

Instead, the framework handles the request lifecycle.

Simplified:

```
HTTP Request
     |
     v
Tomcat
     |
     v
DispatcherServlet
     |
     v
Find matching handler
     |
     v
HelloController.hello()
```

This is an important example of **framework-controlled execution**.

---

# 65\. Why Understanding Spring Internals Matters

A beginner may think:

```
@GetMapping("/hello")
```

is magic.

It isn't.

There is a large framework underneath it:

```
HTTP
 ↓
Servlet API
 ↓
Tomcat
 ↓
DispatcherServlet
 ↓
Handler Mapping
 ↓
Controller
 ↓
Method
```

Understanding this makes debugging much easier.

---

# 66\. What If the Endpoint Doesn't Work?

A useful debugging checklist:

```
1. Is the application running?
        ↓
2. Is Tomcat listening?
        ↓
3. Am I using the correct port?
        ↓
4. Is the URL correct?
        ↓
5. Is the HTTP method correct?
        ↓
6. Was the controller discovered?
        ↓
7. Is @RestController present?
        ↓
8. Is @GetMapping correct?
        ↓
9. Is the package structure correct?
        ↓
10. Is there another configuration/error?
```

---

# 67\. Connection Refused vs 404

This distinction is extremely useful in interviews and debugging.

## Connection Refused

Example:

```
localhost:8080
```

and nothing is listening on 8080.

Possible cause:

```
Spring Boot isn't running
OR
Wrong port
OR
Network/server issue
```

The request may not reach the application at all.

---

## 404 Not Found

The server is reachable, but the requested resource/route was not found.

Example:

```
GET /does-not-exist
```

Possible result:

```
404 Not Found
```

Think:

```
Connection refused
    ↓
Couldn't connect to server

404
    ↓
Connected to server,
but requested route/resource wasn't found
```

---

# 68\. Important Annotation Cheat Sheet

| Annotation | Purpose |
| --- | --- |
| `@SpringBootApplication` | Main Spring Boot configuration/startup annotation |
| `@RestController` | REST/web controller |
| `@Controller` | MVC controller |
| `@GetMapping` | Maps HTTP GET requests |
| `@PostMapping` | Maps HTTP POST requests |
| `@PutMapping` | Maps HTTP PUT requests |
| `@PatchMapping` | Maps HTTP PATCH requests |
| `@DeleteMapping` | Maps HTTP DELETE requests |
| `@Service` | Marks service-layer component |
| `@Repository` | Marks persistence-layer component |
| `@Component` | Generic Spring-managed component |

---

# 69\. Common Interview Questions — Networking

## Q1. What is localhost?

**Answer:**

`localhost` is a hostname that refers to the local machine. For IPv4, it commonly resolves to a loopback address such as `127.0.0.1`.

---

## Q2. What is `127.0.0.1`?

**Answer:**

It is an IPv4 loopback address that routes traffic back to the same machine.

---

## Q3. What is DNS?

**Answer:**

DNS, or Domain Name System, maps domain names to IP addresses and supports other DNS record lookups.

---

## Q4. Why do we need DNS?

**Answer:**

Humans use convenient domain names, while network communication uses IP addresses. DNS provides the mapping between them.

---

## Q5. What is a port?

**Answer:**

A port identifies a network service endpoint on a host and allows multiple network services to use the same IP address.

---

## Q6. What are the default ports for HTTP and HTTPS?

**Answer:**

```
HTTP  → 80
HTTPS → 443
```

---

## Q7. Is 8080 an HTTP standard port?

**Answer:**

No. `8080` is a commonly used alternative HTTP/application port and is the common default for Spring Boot's embedded web server.

---

## Q8. What is a reverse proxy?

**Answer:**

A reverse proxy is a server-side intermediary that accepts client requests and forwards them to backend servers. It can also provide TLS termination, routing, load balancing, and other infrastructure capabilities.

---

# 70\. Common Interview Questions — Spring Boot

## Q9. What is Spring Initializr?

**Answer:**

Spring Initializr is a project-generation service that creates a Spring Boot project with the selected build tool, Java version, Spring Boot version, and dependencies.

---

## Q10. What is Maven?

**Answer:**

Maven is a build automation and dependency management tool commonly used for Java projects.

---

## Q11. What is `pom.xml`?

**Answer:**

`pom.xml` is Maven's Project Object Model file. It defines project metadata, dependencies, plugins, and build configuration.

---

## Q12. What is a dependency?

**Answer:**

A dependency is an external library or module required by an application.

---

## Q13. Why do we use Maven?

**Answer:**

Maven automates dependency management and standardizes tasks such as compilation, testing, packaging, and plugin execution.

---

## Q14. What is a JAR?

**Answer:**

JAR stands for Java Archive. It packages Java classes/resources and, in Spring Boot, can be used to distribute an executable application.

---

## Q15. What is an embedded server?

**Answer:**

An embedded server is a web server/container packaged with the application so the application can start the server itself rather than requiring a separately installed external server.

---

# 71\. Common Interview Questions — Spring Boot Startup

## Q16. What does `SpringApplication.run()` do?

**Answer:**

It bootstraps the Spring application, creates the application context, performs configuration and bean setup, and for a web application starts the embedded web server.

---

## Q17. What is `@SpringBootApplication`?

**Answer:**

It is a composed Spring Boot annotation that combines `@Configuration`, `@EnableAutoConfiguration`, and `@ComponentScan`.

---

## Q18. What is auto-configuration?

**Answer:**

Spring Boot's auto-configuration mechanism attempts to configure appropriate application infrastructure based on the application's dependencies, configuration, and environment.

---

## Q19. What is component scanning?

**Answer:**

Component scanning allows Spring to discover classes annotated as Spring components, such as `@Component`, `@Service`, `@Repository`, and controllers, and register them with the application context.

---

## Q20. Why is package structure important?

**Answer:**

Because component scanning has a defined scope. If your controller is outside the packages being scanned, Spring may not discover it, resulting in mapping/bean-related problems.

---

# 72\. Common Interview Questions — REST Controller

## Q21. What is `@RestController`?

**Answer:**

`@RestController` marks a class as a controller whose methods generally return data directly in the HTTP response body. It combines `@Controller` and `@ResponseBody`.

---

## Q22. What does `@GetMapping` do?

**Answer:**

It maps HTTP GET requests matching a specified path to a controller method.

---

## Q23. Does the method name determine the endpoint?

**Answer:**

No.

```
@GetMapping("/hello")
public String anything() {
    return "Hello";
}
```

The endpoint is:

```
/hello
```

not `/anything`.

---

## Q24. Can two methods have the same `@GetMapping`?

Not with the same request mapping in a way that creates an ambiguous mapping.

For example:

```
@GetMapping("/hello")
method1()

@GetMapping("/hello")
method2()
```

would create an ambiguous mapping and Spring cannot determine which method should handle the request.

---

# 73\. Common Interview Questions — Configuration

## Q25. How do you change Spring Boot's default port?

Add:

```
server.port=9090
```

to:

```
src/main/resources/application.properties
```

---

## Q26. Can the port be changed without changing Java code?

Yes.

For example:

```
server.port=9090
```

This is one benefit of externalized configuration.

---

## Q27. What happens if you access port 8080 after changing the application to 9090?

If nothing else is listening on 8080, the connection will generally fail because the Spring Boot application is now listening on 9090.

---

# 74\. Scenario-Based Interview Questions

## Scenario 1

> You start Spring Boot successfully but `localhost:8080/hello` doesn't work. What do you check?

Answer:

```
Is application running?
       ↓
Correct port?
       ↓
Correct URL?
       ↓
Correct HTTP method?
       ↓
@RestController present?
       ↓
@GetMapping correct?
       ↓
Controller discovered?
       ↓
Package scanning correct?
       ↓
Any startup/runtime errors?
```

---

## Scenario 2

> Your application starts successfully, but `/hello` returns 404. What could be wrong?

Possible reasons:

- No mapping for `/hello`
- Incorrect path
- Wrong HTTP method
- Controller not discovered
- Controller annotation missing
- Component scanning issue

---

## Scenario 3

> Your application doesn't start because port 8080 is already in use. What happened?

Another process is already listening on port 8080.

Possible solutions:

```
Stop the other process
OR
Change Spring Boot's port
```

For example:

```
server.port=9090
```

---

## Scenario 4

> Why can two applications not normally listen on the exact same IP/port combination?

Because the operating system needs to uniquely associate incoming connections with a listening socket. A second process generally cannot bind to the same address/port combination unless special socket options/configurations allow it.

---

# 75\. The Most Important Production Diagram

Memorize this:

```
                      USER
                       |
                       v
                  WEB BROWSER
                       |
                       | HTTPS
                       v
                  DNS LOOKUP
                       |
                       v
                 SERVER IP
                       |
                       | :443
                       v
              +----------------+
              | Reverse Proxy  |
              +-------+--------+
                      |
                      | Internal routing
                      v
              +----------------+
              | Spring Boot    |
              | Application    |
              | Embedded Tomcat|
              +-------+--------+
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

---

# 76\. The Most Important Local Diagram

```
+------------------+
|     Browser      |
|     CLIENT       |
+--------+---------+
         |
         | HTTP GET
         | localhost:8080/hello
         v
+------------------+
|   Your Computer  |
|                  |
| Spring Boot App  |
|                  |
| Embedded Tomcat  |
|       :8080      |
+--------+---------+
         |
         v
   DispatcherServlet
         |
         v
   HelloController
         |
         v
      hello()
         |
         v
    "Hello World"
```

---

# 77\. Local vs Production — Interview Answer

If asked:

> Explain the difference between calling a Spring Boot application locally and calling a production application.

Say:

> Locally, the browser can connect directly to a Spring Boot application running on the same machine, typically through `localhost` and a development port such as 8080. In production, the client usually accesses a domain name. DNS resolves the domain to an IP address, HTTPS commonly uses port 443, and the request may pass through a reverse proxy or load balancer before reaching one of the Spring Boot application instances on an internal port.

That's a strong interview answer.

---

# 78\. Why Spring Boot Feels Like "Magic"

As a beginner:

```
@RestController
@GetMapping("/hello")
```

looks magical.

But underneath:

```
HTTP
  ↓
TCP/TLS/network
  ↓
Web Server
  ↓
Servlet Container
  ↓
DispatcherServlet
  ↓
Spring MVC
  ↓
Handler Mapping
  ↓
Controller Bean
  ↓
Java Method
```

Spring Boot hides infrastructure so you can focus on application/business logic.

---

# 79\. But Should You Learn the Hidden Layers?

**Absolutely.**

You don't need to implement Tomcat yourself.

But you should understand:

```
What Tomcat does
What a Servlet is
What DispatcherServlet does
What a Controller does
How request mapping works
How Spring creates/manages components
```

Why?

Because debugging becomes much easier.

---

# 80\. Example Debugging Thought Process

Suppose:

```
GET /hello
```

returns 404.

Don't randomly change annotations.

Think through the request:

```
Did request reach the machine?
       ↓
Did it reach the correct port?
       ↓
Did Tomcat receive it?
       ↓
Did Spring receive it?
       ↓
Was controller discovered?
       ↓
Is mapping registered?
       ↓
Does GET /hello match?
       ↓
Does controller method execute?
```

This is professional debugging.

---

# 81\. Interview Trap: "Spring Boot Starts Tomcat"

A better explanation is:

> In a typical Spring Boot servlet web application using Tomcat, Spring Boot starts and configures the embedded Tomcat web server as part of application startup.

Don't oversimplify it to:

> Spring Boot is Tomcat.

They are completely different things.

```
Spring Boot
    ↓
Application bootstrapping/configuration

Tomcat
    ↓
Servlet container/web server
```

---

# 82\. Interview Trap: "Spring Boot Is a Server"

Better:

> Spring Boot is a framework/project for building Spring applications. A typical Spring Boot web application can include and start an embedded web server such as Tomcat.

---

# 83\. Interview Trap: "localhost Is an IP Address"

Strictly:

```
localhost
```

is a hostname.

It commonly resolves to:

```
127.0.0.1
```

So:

```
localhost ≠ literally an IP address
```

---

# 84\. Interview Trap: "Every Computer Has a Unique IP"

This statement is too simplistic.

Modern networks commonly use:

- Private IP addresses
- Public IP addresses
- NAT
- IPv4
- IPv6
- Dynamic addressing

So don't memorize:

> Every device has one unique public IP.

Instead:

> Hosts/interfaces can have IP addresses, and public/private addressing depends on the network architecture.

---

# 85\. Interview Trap: "DNS Server Gives the Browser the Server"

Better:

> DNS resolves a domain name to DNS records, commonly including an IP address used to locate the destination. The actual request may then pass through proxies, load balancers, CDNs, gateways, and other infrastructure before reaching the application.

---

# 86\. Interview Trap: "Spring Web = Tomcat"

Better:

> Spring Web provides the Spring web/MVC functionality and, in the standard Spring Boot servlet setup, the corresponding starter brings in an embedded servlet container such as Tomcat.

---

# 87\. Interview Trap: "JAR Is Always Better Than WAR"

Better:

> Executable JAR packaging is common and convenient in modern Spring Boot applications because the application can run with an embedded server, while WAR packaging remains supported for deployments to externally managed servlet containers.

---

# 88\. Rapid Revision — 60 Seconds

```
localhost
    ↓
Current machine

127.0.0.1
    ↓
IPv4 loopback

DNS
    ↓
Domain → DNS records/IP

Port
    ↓
Network service endpoint

HTTP
    ↓
Port 80 by default

HTTPS
    ↓
Port 443 by default

Spring Boot
    ↓
Common default web port 8080

Reverse Proxy
    ↓
Front door to backend services

Maven
    ↓
Build + dependency management

pom.xml
    ↓
Maven project configuration

Spring Initializr
    ↓
Generate Spring Boot project

JAR
    ↓
Common Spring Boot packaging

Spring Web
    ↓
Build web/API applications

Tomcat
    ↓
Servlet container

@SpringBootApplication
    ↓
Configuration + auto-configuration + component scanning

@RestController
    ↓
REST controller

@GetMapping
    ↓
Map GET request to method

application.properties
    ↓
Application configuration
```

---

# 89\. Interview Questions You Should Be Able to Answer Without Notes

## Networking

- [ ] What is client-server architecture?
- [ ] What is localhost?
- [ ] What is `127.0.0.1`?
- [ ] What is DNS?
- [ ] Why do we need DNS?
- [ ] What is an IP address?
- [ ] What is a port?
- [ ] IP vs port?
- [ ] What is port 80?
- [ ] What is port 443?
- [ ] Why does Spring Boot commonly use 8080?
- [ ] What is a reverse proxy?
- [ ] Why use a reverse proxy?
- [ ] Local vs production request flow?

## Maven / Project Setup

- [ ] What is Spring Initializr?
- [ ] What is Maven?
- [ ] What is a dependency?
- [ ] What is `pom.xml`?
- [ ] What is a JAR?
- [ ] JAR vs WAR?
- [ ] What is an embedded server?
- [ ] Why use an embedded server?

## Spring Boot

- [ ] What does `SpringApplication.run()` do?
- [ ] What is `@SpringBootApplication`?
- [ ] What is auto-configuration?
- [ ] What is component scanning?
- [ ] Why is package structure important?
- [ ] What is Spring Web?
- [ ] What is Tomcat?
- [ ] How does Spring Boot start Tomcat?

## REST

- [ ] What is `@RestController`?
- [ ] What is `@GetMapping`?
- [ ] Does method name determine endpoint?
- [ ] How does a request reach a controller?
- [ ] What happens if no mapping exists?
- [ ] Why might you get 404?
- [ ] Connection refused vs 404?
- [ ] How do you change the Spring Boot port?

---

# 90\. Final Mental Model

The most important thing from this lecture is not memorizing annotations.

Understand this:

```
                       INTERNET
                           |
                           v
                         DNS
                           |
                           v
                    SERVER IP :443
                           |
                           v
                    Reverse Proxy
                           |
                           v
                Spring Boot Application
                           |
                           v
                    Embedded Tomcat
                           |
                           v
                  DispatcherServlet
                           |
                           v
                     Controller
                           |
                           v
                      Java Method
```

And locally:

```
Browser
   |
   | http://localhost:8080/hello
   v
Your Computer
   |
   v
Spring Boot
   |
   v
Embedded Tomcat
   |
   v
DispatcherServlet
   |
   v
HelloController
   |
   v
hello()
   |
   v
"Hello World"
```

The key philosophy is:

> **Spring Boot hides infrastructure complexity, but a professional Spring developer should understand the infrastructure underneath the abstractions.**
