# Maven, JAR, Dependencies & Build Lifecycle

> **Goal:** Understand how Java applications are packaged, how dependencies are managed, how Maven builds a project, and how Maven is used in real Spring Boot projects.

---

# 1\. Big Picture: From Java Code to a Running Application

Before understanding Maven, first understand what happens to Java code.

```
                COMPILE
Java Source  ───────────────>  Bytecode
  .java                         .class
                                  │
                                  │ JVM
                                  ▼
                             Running App
```

For a simple Java program:

```
Demo.java
   │
   │ javac
   ▼
Demo.class
   │
   │ java Demo
   ▼
JVM executes bytecode
```

### Important

`.java` = source code written by the developer.

`.class` = compiled Java bytecode.

`.jar` = package containing compiled classes + resources + metadata.

---

# 2\. What is a JAR?

## Definition

**JAR = Java ARchive**

A JAR is a package format used to bundle Java classes, resources, and metadata into a single file.

It is based on the ZIP archive format.

Example:

```
calculator.jar
│
├── META-INF/
│   └── MANIFEST.MF
│
├── com/
│   └── example/
│       ├── Calculator.class
│       ├── MathUtils.class
│       └── User.class
│
└── application.properties
```

Instead of sending:

```
Calculator.class
MathUtils.class
User.class
application.properties
...
```

we can send:

```
calculator.jar
```

---

# 3\. Why Do We Need JAR Files?

Imagine an application containing:

```
500 Java classes
50 configuration files
20 images
10 JSON files
5 XML files
```

Sending all these files individually is inconvenient.

A JAR provides:

```
Many Files
   │
   ▼
┌───────────────────────────┐
│       application.jar     │
│                           │
│  Classes                  │
│  Resources                │
│  Metadata                 │
│  Package structure        │
└───────────────────────────┘
```

### Benefits

- Easy distribution
- Preserves directory/package structure
- Bundles many files together
- Convenient deployment
- Can be executable
- Can be used as a dependency/library

---

# 4\. Library JAR vs Executable JAR

This is an important interview topic.

## Library JAR

A library JAR is intended to be **used by another application**.

Example:

```
calculator.jar
database-driver.jar
logging-library.jar
```

Another application can use it:

```
import com.example.Calculator;
```

The library itself doesn't necessarily need to be directly runnable.

---

## Executable JAR

An executable JAR is an application that can be launched.

For example:

```
java -jar my-app.jar
```

It must have an appropriate application entry point.

A simple Java executable JAR can use:

```
public static void main(String[] args) {
    System.out.println("Application started");
}
```

Spring Boot executable JARs go a step further: the Spring Boot Maven/Gradle plugin creates a runnable JAR layout and launcher so the application can start with:

```
java -jar app.jar
```

---

# 5\. Thin JAR vs Fat/Executable JAR

Another useful interview distinction.

## Thin JAR

Contains primarily your application's own classes/resources.

```
my-app.jar
│
├── MyController.class
├── UserService.class
└── UserRepository.class
```

External dependencies are expected to be available separately.

---

## Fat JAR / Uber JAR

Contains your application plus its dependencies.

Conceptually:

```
my-app.jar
│
├── Your Application
│
├── Spring
├── Jackson
├── Tomcat
├── Database Driver
└── Other Dependencies
```

This is especially useful for Spring Boot deployment.

```
java -jar application.jar
```

You don't have to manually construct a huge classpath containing every dependency.

> **Interview note:** "Fat JAR" and "Uber JAR" are commonly used terms. A Spring Boot executable JAR has a particular internal layout rather than simply being a normal ZIP with all JAR contents flattened together.

---

# 6\. What is the Classpath?

The **classpath** tells Java where it should look for classes and resources.

Suppose:

```
import com.mysql.cj.jdbc.Driver;
```

Your application needs the MySQL driver class.

The JVM needs to know where that class is located.

Conceptually:

```
Application
     │
     │ needs MySQL Driver
     ▼
Classpath
     │
     ▼
mysql-connector-j.jar
     │
     ▼
com/mysql/cj/jdbc/Driver.class
```

If the required class cannot be found, you can get errors such as:

```
ClassNotFoundException
```

or related class-loading/linkage errors depending on the situation.

---

# 7\. Classpath vs Dependency

These concepts are related but not identical.

### Dependency

A library your application needs.

Example:

```
Spring Web
MySQL Driver
JUnit
Jackson
```

### Classpath

The collection of locations from which Java can load classes/resources during a particular execution.

Maven helps manage dependencies and ultimately contributes those dependencies to the appropriate classpaths.

```
pom.xml
   │
   ▼
Maven resolves dependencies
   │
   ▼
JARs downloaded
   │
   ▼
Classpath configured
   │
   ▼
JVM can load required classes
```

---

# 8\. The Problem Before Maven

Imagine you need:

```
Spring
MySQL Driver
Jackson
Logging
JUnit
```

Without a dependency-management tool, you might manually download:

```
spring-x.jar
mysql-x.jar
jackson-x.jar
logging-x.jar
junit-x.jar
```

Then put them somewhere in your project.

But this creates problems.

---

# 9\. Problem #1 — Manual Dependency Management

You have to:

```
Search
  ↓
Download
  ↓
Copy JAR
  ↓
Configure classpath
  ↓
Repeat
```

This becomes painful when an application has dozens or hundreds of dependencies.

---

# 10\. Problem #2 — Version Conflicts

Suppose:

```
Application
   │
   ├── Library A → Framework 6
   │
   └── Library B → Framework 7
```

Now the application may have incompatible versions.

This is called a **dependency/version conflict**.

Maven provides dependency mediation and dependency management mechanisms to help resolve such situations.

---

# 11\. Problem #3 — Transitive Dependencies

This is extremely important.

Suppose your application directly depends on:

```
Library A
```

But Library A itself needs:

```
Library B
```

And Library B needs:

```
Library C
```

Then:

```
Your Application
      │
      ▼
   Library A
      │
      ▼
   Library B
      │
      ▼
   Library C
```

B and C are **transitive dependencies** of your application.

Maven can resolve these automatically.

---

# 12\. Problem #4 — Collaboration

Suppose you send your application to another developer.

Without dependency management:

```
"Here are 37 JAR files.
Please download these versions and configure them."
```

With Maven:

```
"Clone the project and run Maven."
```

The dependency declarations live in:

```
pom.xml
```

So Maven knows what is required.

---

# 13\. What is Maven?

**Apache Maven** is a build automation and dependency management tool primarily used in Java projects.

Maven helps with:

- Dependency management
- Compilation
- Testing
- Packaging
- Project structure
- Plugin execution
- Artifact installation
- Artifact deployment

Think of Maven as:

```
                 MAVEN
                    │
       ┌────────────┼────────────┐
       ▼            ▼            ▼
 Dependencies    Build        Packaging
       │            │            │
       ▼            ▼            ▼
 Download       Compile       JAR/WAR
 Resolve        Test
```

---

# 14\. Maven's Most Important Job

If you remember only one thing:

> **Maven automates the build process and manages project dependencies.**

For example, instead of manually downloading Spring libraries, you declare:

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

Maven handles the resolution process.

---

# 15\. Maven Standard Directory Structure

Maven follows a conventional project structure.

```
my-project/
│
├── pom.xml
│
├── src/
│   │
│   ├── main/
│   │   ├── java/
│   │   │   └── com/example/
│   │   │       └── App.java
│   │   │
│   │   └── resources/
│   │       └── application.properties
│   │
│   └── test/
│       ├── java/
│       │   └── com/example/
│       │       └── AppTest.java
│       │
│       └── resources/
│
└── target/
```

---

# 16\. Important Maven Folders

## `src/main/java`

Contains production Java source code.

Example:

```
src/main/java/com/example/UserService.java
```

---

## `src/main/resources`

Contains application resources.

Examples:

```
application.properties
application.yml
static/
templates/
JSON files
XML files
```

---

## `src/test/java`

Contains test source code.

Examples:

```
UserServiceTest.java
OrderServiceTest.java
```

---

## `src/test/resources`

Resources required specifically by tests.

---

## `target/`

Contains generated build output.

Example:

```
target/
│
├── classes/
├── test-classes/
├── surefire-reports/
└── my-app.jar
```

> **Important:** `target/` is generated. You normally don't manually write application source code there.

---

# 17\. What is `pom.xml`?

`pom.xml` stands for:

> **Project Object Model**

It is Maven's central configuration file.

```
                 pom.xml
                    │
       ┌────────────┼─────────────┐
       ▼            ▼             ▼
 Project Info   Dependencies    Build Config
       │            │             │
       ▼            ▼             ▼
   groupId       Spring        Plugins
   artifactId    MySQL         Packaging
   version       JUnit         Compiler
```

---

# 18\. Basic POM Structure

```
<project>

    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>my-app</artifactId>
    <version>1.0.0</version>

    <dependencies>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

    </dependencies>

</project>
```

---

# 19\. Maven Coordinates — GAV

Every Maven artifact is identified using coordinates.

The three most important coordinates are:

```
G = groupId
A = artifactId
V = version
```

Together:

```
groupId : artifactId : version
```

Example:

```
com.example : payment-service : 1.0.0
```

---

# 20\. `groupId`

Usually identifies the organization/project namespace.

Example:

```
<groupId>com.amazon</groupId>
```

or:

```
<groupId>in.coderarmy</groupId>
```

Conventionally, reverse-domain notation is used.

For:

```
coderarmy.in
```

we might use:

```
in.coderarmy
```

---

# 21\. `artifactId`

Identifies the particular project/artifact.

Example:

```
<artifactId>payment-service</artifactId>
```

The resulting artifact might be:

```
payment-service-1.0.0.jar
```

---

# 22\. `version`

Identifies the version of the artifact.

Examples:

```
1.0.0
1.1.0
2.0.0
```

Development versions may use:

```
1.0.0-SNAPSHOT
```

---

# 23\. SNAPSHOT vs RELEASE

### SNAPSHOT

Example:

```
1.0.0-SNAPSHOT
```

Means the artifact represents an ongoing development version.

It is not intended to represent a fixed final release.

### Release

Example:

```
1.0.0
```

Normally represents a stable published version.

```
Development
     │
     ▼
1.0.0-SNAPSHOT
     │
     │ stable
     ▼
1.0.0
     │
     ▼
1.1.0
```

> **Interview trap:** SNAPSHOT does not simply mean "contains bugs." It means a development version whose artifact may change over time.

---

# 24\. `<packaging>`

Defines the packaging type.

Common values:

```
<packaging>jar</packaging>
```

or:

```
<packaging>war</packaging>
```

If omitted, Maven defaults to:

```
jar
```

Spring Boot applications commonly use JAR packaging.

---

# 25\. `<properties>`

Properties allow centralized configuration.

Example:

```
<properties>
    <java.version>21</java.version>
</properties>
```

A property can be referenced as:

```
${java.version}
```

Example:

```
<maven.compiler.source>${java.version}</maven.compiler.source>
```

This avoids repeating the same value.

---

# 26\. Dependencies in `pom.xml`

Example:

```
<dependencies>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <dependency>
        <groupId>com.mysql</groupId>
        <artifactId>mysql-connector-j</artifactId>
    </dependency>

</dependencies>
```

Maven reads these declarations and resolves the required artifacts.

---

# 27\. What Happens When Maven Sees a Dependency?

Suppose:

```
<dependency>
    <groupId>com.mysql</groupId>
    <artifactId>mysql-connector-j</artifactId>
    <version>...</version>
</dependency>
```

Conceptually:

```
             pom.xml
                │
                ▼
       Maven reads dependency
                │
                ▼
       Check local repository
                │
        ┌───────┴────────┐
        │                │
      Found           Not Found
        │                │
        ▼                ▼
      Use        Search configured
                 remote repository
                        │
                        ▼
                     Download
                        │
                        ▼
                 Save in .m2
                        │
                        ▼
                      Build
```

---

# 28\. Maven Repositories

A repository is a location where Maven artifacts are stored.

The three important categories are:

```
1. Local Repository
2. Central Repository
3. Other Remote Repositories
```

---

# 29\. Local Repository

Usually:

```
~/.m2/repository
```

Windows example:

```
C:\Users\<username>\.m2\repository
```

Linux/macOS:

```
~/.m2/repository
```

It acts as Maven's local cache.

Example:

```
~/.m2/repository/
└── org/
    └── springframework/
        └── boot/
            └── spring-boot/
```

---

# 30\. Why Does Maven Use `.m2`?

Suppose you build five projects and all need the same dependency:

```
spring-core.jar
```

Maven doesn't need to download it separately every time.

Instead:

```
                 Internet
                    │
                    ▼
              Download once
                    │
                    ▼
             ~/.m2/repository
                    │
        ┌───────────┼───────────┐
        ▼           ▼           ▼
    Project A   Project B   Project C
```

This saves bandwidth and time.

---

# 31\. Maven Central

Maven Central is the primary public repository for a huge number of Java artifacts.

If the required dependency isn't available locally, Maven can retrieve it from configured remote repositories, commonly Maven Central.

```
pom.xml
   │
   ▼
Local Repository
   │
   │ not found
   ▼
Remote Repository
   │
   ▼
Download artifact
   │
   ▼
Local Repository
```

---

# 32\. Private Repositories

Companies often maintain internal repositories.

Popular technologies include:

- JFrog Artifactory
- Sonatype Nexus

Example:

```
Developer
    │
    ▼
Maven
    │
    ▼
Company Repository
    │
    ├── Internal libraries
    ├── Approved external libraries
    └── Cached public libraries
```

Why?

- Internal/proprietary artifacts
- Security controls
- Dependency governance
- Faster internal builds
- Centralized artifact management

---

# 33\. `mvn install` vs `mvn deploy`

Very common interview question.

### `mvn install`

Installs your artifact into your **local Maven repository**.

```
Project
   │
   ▼
target/my-library.jar
   │
   ▼
~/.m2/repository
```

Other projects on the **same machine** can then use it.

### `mvn deploy`

Publishes the artifact to a configured **remote repository**.

```
Project
   │
   ▼
Maven
   │
   ▼
Company Remote Repository
```

Other developers/build systems can retrieve it.

---

# 34\. Maven Lifecycle

Maven provides predefined build lifecycles.

The most important one for everyday Java/Spring development is the:

> **Default lifecycle**

Important phases include:

```
validate
   ↓
compile
   ↓
test
   ↓
package
   ↓
verify
   ↓
install
   ↓
deploy
```

---

# 35\. The Golden Maven Rule

This is one of the most important concepts.

> **When you run a Maven phase, Maven executes that phase and all earlier phases in the same lifecycle.**

For example:

```
mvn package
```

effectively goes through:

```
validate
   ↓
compile
   ↓
test
   ↓
package
```

You don't need to manually run:

```
mvn validate
mvn compile
mvn test
mvn package
```

---

# 36\. `mvn validate`

Checks that the project is valid enough to begin the build.

```
mvn validate
```

Think:

```
"Is my Maven project configuration valid?"
```

---

# 37\. `mvn compile`

Compiles production Java source.

```
src/main/java
       │
       │ Maven Compiler Plugin
       ▼
target/classes
       │
       ├── UserService.class
       ├── UserController.class
       └── OrderService.class
```

Command:

```
mvn compile
```

---

# 38\. `mvn test`

Compiles and runs tests during the test phase.

Conceptually:

```
src/main/java
      │
      ▼
Compile production code
      │
      ▼
src/test/java
      │
      ▼
Compile tests
      │
      ▼
Run tests
```

Command:

```
mvn test
```

If a required test fails, the build normally fails at that stage.

---

# 39\. `mvn package`

Creates the distributable artifact.

For a JAR project:

```
target/
└── my-app-1.0.0.jar
```

Command:

```
mvn package
```

Because of lifecycle ordering:

```
mvn package
```

runs earlier phases first.

---

# 40\. `mvn verify`

The `verify` phase allows additional verification such as integration tests or quality checks configured through plugins.

```
mvn verify
```

Think:

> "Does the built project satisfy the checks configured for this build?"

The exact checks depend on the project's plugins/configuration.

---

# 41\. `mvn install`

Runs the earlier default lifecycle phases and then installs the artifact into your local repository.

```
mvn install
```

Result:

```
target/
   │
   └── my-app-1.0.0.jar
             │
             ▼
~/.m2/repository/
   └── com/example/my-app/1.0.0/
```

---

# 42\. `mvn deploy`

Publishes the artifact to a configured remote repository.

```
mvn deploy
```

Typical enterprise flow:

```
Developer
    │
    ▼
mvn deploy
    │
    ▼
Build + Test + Package
    │
    ▼
Company Artifact Repository
    │
    ├── Developer A
    ├── Developer B
    └── CI/CD Server
```

---

# 43\. `clean` Lifecycle

`clean` is a separate Maven lifecycle.

Its primary purpose is to remove generated build output.

```
mvn clean
```

Typically removes:

```
target/
```

So:

```
Before:

target/
├── classes/
├── test-classes/
└── app.jar

mvn clean

After:

target/
└── deleted
```

---

# 44\. Why Use `mvn clean`?

Suppose old compiled files are causing confusing behavior.

You can do:

```
mvn clean
```

and then rebuild.

This gives you a fresh build environment.

---

# 45\. The Famous Command: `mvn clean install`

Very common in Java projects.

```
mvn clean install
```

Conceptually:

```
clean
  │
  ▼
Delete target/
  │
  ▼
validate
  │
  ▼
compile
  │
  ▼
test
  │
  ▼
package
  │
  ▼
verify
  │
  ▼
install into .m2
```

This is frequently used when you want to perform a fresh local build and make the resulting artifact available to other local Maven projects.

---

# 46\. Important: `clean` Is NOT Part of Default Lifecycle

This is an interview favorite.

These are separate lifecycles:

```
Clean Lifecycle
     │
     └── clean

Default Lifecycle
     │
     ├── validate
     ├── compile
     ├── test
     ├── package
     ├── verify
     ├── install
     └── deploy

Site Lifecycle
     │
     └── Documentation/report generation
```

Therefore:

```
mvn clean package
```

runs:

```
clean lifecycle
        +
default lifecycle through package
```

---

# 47\. Maven Plugins

Maven itself relies heavily on plugins.

Examples of responsibilities:

```
Compiler Plugin
      ↓
Compile Java

Surefire Plugin
      ↓
Run unit tests

Spring Boot Maven Plugin
      ↓
Package/run Spring Boot application
```

A useful mental model is:

```
Maven Lifecycle
      │
      ▼
Phase
      │
      ▼
Plugin Goal
      │
      ▼
Actual work
```

---

# 48\. Lifecycle Phase vs Plugin Goal

These are NOT the same thing.

### Phase

Example:

```
compile
package
test
```

### Plugin Goal

Example:

```
compiler:compile
surefire:test
spring-boot:run
```

Think:

```
Maven Phase
    │
    ▼
Plugin execution
    │
    ▼
Goal
```

This distinction is very useful in interviews.

---

# 49\. Maven Archetype

A Maven **Archetype** is a project template.

It generates a basic Maven project structure for you.

Conceptually:

```
Archetype Template
       │
       ▼
Generate Project
       │
       ▼
pom.xml
src/main/java
src/test/java
sample classes
```

Example:

```
maven-archetype-quickstart
```

Historically, this has been used to generate a basic Maven Java project.

---

# 50\. Maven Archetype vs Spring Initializr

These are often confused.

### Maven Archetype

General-purpose Maven project template mechanism.

### Spring Initializr

A project generator designed specifically for Spring applications.

For Spring Boot development, you commonly use:

```
start.spring.io
```

to choose:

```
Java
Maven
Spring Boot
Dependencies
Packaging
Java version
```

and generate the project.

---

# 51\. Maven + Spring Boot

A typical Spring Boot project looks like:

```
Spring Boot Application
        │
        ▼
      Maven
        │
        ├── Dependencies
        ├── Compilation
        ├── Testing
        ├── Packaging
        └── Plugins
```

Example dependency:

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

Maven resolves the starter and its required dependencies.

---

# 52\. Spring Boot Parent POM

Spring Boot projects may use:

```
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>...</version>
</parent>
```

The parent can provide useful defaults and dependency management.

For example, Spring Boot manages compatible versions of many dependencies so developers don't have to specify every individual version manually.

---

# 53\. Dependency Management vs Dependency Declaration

Important distinction.

### Dependency Declaration

Says:

> "My project needs this dependency."

Example:

```
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>
```

### Dependency Management

Can control versions/configuration of dependencies without necessarily adding those dependencies to the project.

Example concept:

```
<dependencyManagement>
    ...
</dependencyManagement>
```

Spring Boot makes heavy use of dependency management.

---

# 54\. Maven Dependency Scope

Dependency scope determines where a dependency is available.

Common scopes:

```
compile
provided
runtime
test
system
import
```

The most important beginner-level ones are:

## `compile`

Available for normal compilation and runtime.

This is the default scope in Maven when applicable.

```
<scope>compile</scope>
```

Usually you don't need to explicitly write it.

---

## `test`

Available only for test compilation/execution.

Example:

```
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter</artifactId>
    <scope>test</scope>
</dependency>
```

Your production application doesn't need JUnit at runtime.

---

# 55\. Example: Complete Maven Flow

Suppose we have:

```
Order Service
```

with:

```
Spring Boot
MySQL
JUnit
```

Our `pom.xml` declares dependencies.

Then:

```
mvn clean package
```

does roughly:

```
1. Delete old target/
          ↓
2. Validate project
          ↓
3. Resolve dependencies
          ↓
4. Compile application
          ↓
5. Compile tests
          ↓
6. Run tests
          ↓
7. Package application
          ↓
8. Produce JAR
```

Result:

```
target/
└── order-service-1.0.0.jar
```

---

# 56\. End-to-End Mental Model

Remember this diagram:

```
                  Developer
                     │
                     │ writes code
                     ▼
              src/main/java
                     │
                     │
                     ▼
                  pom.xml
                     │
             ┌───────┴────────┐
             │                │
             ▼                ▼
       Dependencies         Plugins
             │                │
             ▼                ▼
       Maven Repository    Build Lifecycle
             │                │
             └───────┬────────┘
                     ▼
                  Compile
                     │
                     ▼
                   Test
                     │
                     ▼
                  Package
                     │
                     ▼
               target/app.jar
                     │
                     ▼
               Deploy/Run
```

---

# 57\. Most Important Commands

## Check Maven version

```
mvn -version
```

---

## Compile

```
mvn compile
```

---

## Run tests

```
mvn test
```

---

## Package

```
mvn package
```

---

## Clean build output

```
mvn clean
```

---

## Clean + package

```
mvn clean package
```

---

## Clean + install locally

```
mvn clean install
```

---

## Deploy to configured remote repository

```
mvn deploy
```

---

## Show dependency tree

Extremely useful for debugging dependency conflicts:

```
mvn dependency:tree
```

Example:

```
com.example:order-service
+- org.springframework.boot:spring-boot-starter-web
|  +- spring-web
|  +- spring-webmvc
|  └─ ...
└─ mysql:mysql-connector-j
```

---

## Show effective POM

```
mvn help:effective-pom
```

Useful for understanding the final Maven configuration after inheritance and defaults are applied.

---

# 58\. Maven Dependency Tree

One of the most useful debugging tools.

Suppose:

```
Application
   │
   ├── Library A
   │      └── Common Library 1.0
   │
   └── Library B
          └── Common Library 2.0
```

Run:

```
mvn dependency:tree
```

You can inspect which dependencies are being brought into your application.

---

# 59\. Super POM

Every Maven project ultimately inherits from Maven's built-in **Super POM**.

Conceptually:

```
Super POM
    │
    ▼
Parent POM
    │
    ▼
Your pom.xml
    │
    ▼
Effective POM
```

The Super POM provides Maven defaults.

---

# 60\. Parent POM

A parent POM allows projects to inherit configuration.

Spring Boot projects may use:

```
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>...</version>
</parent>
```

The parent can provide:

- Dependency management
- Plugin configuration
- Default settings
- Build conventions

---

# 61\. Effective POM

The **effective POM** is the resulting Maven configuration after inheritance and configuration are combined.

Think:

```
Super POM
    +
Parent POM
    +
Your POM
    +
Other applicable configuration
    │
    ▼
Effective POM
```

You can inspect it using:

```
mvn help:effective-pom
```

---

# 62\. `pom.xml` vs `target/`

Very important distinction.

| `pom.xml` | `target/` |
| --- | --- |
| Developer configuration | Generated build output |
| Source-controlled | Usually ignored by Git |
| Declares dependencies | Contains compiled artifacts |
| Defines project metadata | Contains JAR/classes |
| Manually maintained | Generated by Maven |

Usually:

```
pom.xml → KEEP
target/ → REGENERATE
```

---

# 63\. `.m2` vs `target`

Another common interview question.

| Location | Purpose |
| --- | --- |
| `target/` | Build output for current project |
| `.m2/repository` | Local Maven dependency/artifact repository |

Example:

```
My Project
│
├── target/
│   └── my-app.jar
│
└── pom.xml

User Home
│
└── .m2/
    └── repository/
        ├── spring/
        ├── junit/
        └── mysql/
```

---

# 64\. JAR vs WAR

## JAR

Java Archive.

Common for:

- Libraries
- Spring Boot applications
- Standalone Java applications

Spring Boot commonly uses executable JARs with embedded server infrastructure.

---

## WAR

Web Application Archive.

Historically common for deploying Java web applications to an externally managed servlet container.

```
WAR
  │
  ▼
External Tomcat
  │
  ▼
Application
```

Modern Spring Boot applications commonly prefer:

```
Executable JAR
    │
    ▼
Embedded Tomcat
    │
    ▼
Application
```

---

# 65\. Maven vs JVM

Don't confuse these.

### JVM

Runs Java bytecode.

```
.class
   │
   ▼
JVM
   │
   ▼
Program execution
```

### Maven

Builds and manages Java projects.

```
.java
   │
   ▼
Maven
   │
   ├── Compile
   ├── Test
   ├── Package
   └── Dependency management
```

Maven ultimately invokes tools/plugins that prepare the artifacts which the JVM can run.

---

# 66\. Maven vs Java Compiler

### `javac`

Compiles Java source:

```
.java → .class
```

### Maven

Automates the overall project build:

```
Dependencies
     ↓
Compile
     ↓
Test
     ↓
Package
     ↓
Install
     ↓
Deploy
```

Maven can invoke the Java compiler through its compiler plugin.

---

# 67\. Maven vs Gradle

Both are build automation/dependency management tools.

### Maven

Uses:

```
pom.xml
```

XML-based configuration.

### Gradle

Usually uses:

```
build.gradle
build.gradle.kts
```

Groovy/Kotlin-based configuration.

Conceptually:

```
Java Project
    │
    ├── Maven
    │     └── pom.xml
    │
    └── Gradle
          └── build.gradle / build.gradle.kts
```

Both can:

- Manage dependencies
- Compile code
- Run tests
- Package applications
- Integrate with CI/CD

---

# 68\. Interview Questions — Beginner Level

## Q1. What is Maven?

**Answer:**

Maven is a Java build automation and dependency management tool. It provides a standard project structure, manages dependencies, runs tests, compiles code, packages applications, and can install/deploy artifacts.

---

## Q2. What is a JAR?

**Answer:**

JAR stands for Java ARchive. It is a package format used to bundle compiled Java classes, resources, and metadata into a single archive.

---

## Q3. What is the difference between `.java`, `.class`, and `.jar`?

**Answer:**

```
.java  → Source code
.class → Compiled bytecode
.jar   → Package containing classes/resources/metadata
```

---

## Q4. What is `pom.xml`?

**Answer:**

`pom.xml` is Maven's Project Object Model file. It contains project metadata, dependencies, properties, build configuration, plugins, and other Maven configuration.

---

## Q5. What are Maven coordinates?

**Answer:**

The primary Maven coordinates are:

```
groupId
artifactId
version
```

Together they identify a Maven artifact.

---

## Q6. What is `groupId`?

**Answer:**

It identifies the organization or namespace that owns the artifact. Reverse-domain naming is commonly used.

Example:

```
com.example
```

---

## Q7. What is `artifactId`?

**Answer:**

It identifies the particular project/artifact.

Example:

```
payment-service
```

---

## Q8. What is a SNAPSHOT version?

**Answer:**

A SNAPSHOT represents a development version that may change before becoming a stable release.

Example:

```
1.0.0-SNAPSHOT
```

---

## Q9. What is the default Maven packaging?

**Answer:**

The default packaging is:

```
jar
```

---

## Q10. What is the local Maven repository?

**Answer:**

It is Maven's local artifact cache, normally located at:

```
~/.m2/repository
```

It stores downloaded dependencies and artifacts installed using `mvn install`.

---

# 69\. Interview Questions — Intermediate

## Q11. What happens when you run `mvn package`?

**Answer:**

Maven executes the required preceding phases in the default lifecycle and then packages the application.

Conceptually:

```
validate
   ↓
compile
   ↓
test
   ↓
package
```

The resulting JAR/WAR is normally placed under:

```
target/
```

---

## Q12. What is the difference between `mvn package` and `mvn install`?

**Answer:**

`package` creates the artifact under `target/`.

`install` goes further and copies the artifact into the local Maven repository.

```
mvn package
    ↓
target/app.jar

mvn install
    ↓
target/app.jar
    +
~/.m2/repository/...
```

---

## Q13. Difference between `install` and `deploy`?

**Answer:**

```
install → local repository
deploy  → configured remote repository
```

---

## Q14. What is a transitive dependency?

**Answer:**

A dependency required by one of your direct dependencies.

```
Application
   ↓
Dependency A
   ↓
Dependency B
```

Here B is a transitive dependency of the application.

---

## Q15. What is a dependency conflict?

**Answer:**

It occurs when different dependencies require incompatible versions of the same library.

Example:

```
Library A → Jackson 2.x
Library B → Jackson 3.x
```

Maven's dependency resolution mechanisms help determine which version is used.

Use:

```
mvn dependency:tree
```

to investigate.

---

## Q16. What is the purpose of `mvn clean`?

**Answer:**

It removes generated build output, typically the `target/` directory, allowing a fresh build.

---

## Q17. Why is `mvn clean install` commonly used?

**Answer:**

It first removes old build output, then performs the default lifecycle through `install`, producing a fresh artifact and placing it in the local Maven repository.

---

## Q18. What is the difference between `target` and `.m2`?

**Answer:**

```
target → output of the current project build

.m2    → local Maven repository/cache
```

---

## Q19. What is a Maven repository?

**Answer:**

A repository stores Maven artifacts such as JARs, POMs, and related metadata.

Common categories include:

- Local repository
- Public remote repositories such as Maven Central
- Private/company remote repositories

---

## Q20. What is a Maven plugin?

**Answer:**

A Maven plugin provides executable functionality for Maven, such as compiling code, running tests, packaging applications, or running Spring Boot.

Examples include compiler and Surefire plugins.

---

# 70\. Interview Questions — Advanced

## Q21. What is the difference between a lifecycle phase and a plugin goal?

**Answer:**

A **lifecycle phase** is a standard step in a Maven lifecycle, such as:

```
compile
test
package
```

A **plugin goal** is a specific operation provided by a plugin.

Examples:

```
compiler:compile
surefire:test
spring-boot:run
```

A phase can have plugin goals bound to it.

---

## Q22. What is the Super POM?

**Answer:**

The Super POM is Maven's built-in default POM configuration. Maven projects inherit default behavior from it.

---

## Q23. What is a Parent POM?

**Answer:**

A Parent POM allows a Maven project to inherit configuration and dependency-management information from another POM.

Spring Boot commonly provides a parent POM:

```
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>...</version>
</parent>
```

---

## Q24. What is an Effective POM?

**Answer:**

The Effective POM is the final Maven configuration resulting from the project's POM, inherited parent configuration, and Maven defaults.

You can inspect it with:

```
mvn help:effective-pom
```

---

## Q25. Why does Maven not require you to manually download transitive dependencies?

**Answer:**

Because Maven reads dependency metadata and recursively resolves required dependencies.

```
Direct Dependency
       ↓
POM metadata
       ↓
Transitive dependencies
       ↓
Download/resolve
       ↓
Classpath/build
```

---

## Q26. What happens if Maven cannot find a dependency?

**Answer:**

Maven attempts to resolve it from the configured repositories. If the artifact cannot be found or downloaded, the build fails with a dependency-resolution error.

---

## Q27. What is dependency scope?

**Answer:**

Dependency scope determines where and when a dependency is available.

Common scopes include:

```
compile
provided
runtime
test
```

For example, a test dependency generally uses:

```
<scope>test</scope>
```

---

## Q28. Why don't we specify versions for every Spring Boot dependency?

**Answer:**

Spring Boot provides dependency management that supplies compatible versions for many dependencies.

This helps avoid manually maintaining a large list of potentially incompatible versions.

---

## Q29. What is a Fat JAR?

**Answer:**

A Fat JAR/Uber JAR packages an application together with its required dependencies so it can be distributed more conveniently.

Spring Boot creates executable JARs designed to run using:

```
java -jar application.jar
```

---

## Q30. What is the purpose of `mvn dependency:tree`?

**Answer:**

It displays the project's dependency graph and is useful for identifying:

- Transitive dependencies
- Duplicate dependencies
- Version conflicts
- Unexpected libraries

---

# 71\. Scenario-Based Interview Questions

## Scenario 1

### Question

You run:

```
mvn package
```

Does Maven only execute the `package` phase?

### Answer

No.

Maven executes the earlier phases in the same lifecycle first.

```
validate
compile
test
package
```

---

# 72\. Scenario 2

### Question

You built a library and want another project on the same laptop to use it.

What command can you use?

### Answer

```
mvn install
```

It installs the artifact into:

```
~/.m2/repository
```

---

# 73\. Scenario 3

### Question

Your teammate also needs your library. What should you use instead of relying on your local `.m2`?

### Answer

Publish it to a shared remote artifact repository using:

```
mvn deploy
```

assuming the project is configured for deployment.

---

# 74\. Scenario 4

### Question

Your application suddenly uses an unexpected version of a dependency.

What command would you run first?

### Answer

```
mvn dependency:tree
```

Then inspect the dependency graph and version mediation.

---

# 75\. Scenario 5

### Question

Your project works on your machine, but you suspect old compiled classes are being used.

What can you try?

### Answer

Run:

```
mvn clean package
```

This removes the generated `target/` directory before rebuilding.

---

# 76\. Scenario 6

### Question

You have:

```
Library A → X 1.0
Library B → X 2.0
```

What problem might occur?

### Answer

A dependency version conflict may occur.

Use:

```
mvn dependency:tree
```

to inspect the resolved dependency graph.

---

# 77\. Scenario 7

### Question

Why can one Maven project use a dependency without manually downloading its JAR?

### Answer

Because Maven reads the dependency declaration from `pom.xml`, resolves the artifact from its repositories, downloads it if necessary, stores it in the local repository, and makes it available to the build.

---

# 78\. Scenario 8

### Question

What is the difference between:

```
mvn clean
```

and:

```
mvn clean install
```

### Answer

```
mvn clean
    → deletes build output

mvn clean install
    → deletes build output
    → builds project
    → tests
    → packages artifact
    → installs artifact locally
```

---

# 79\. Scenario 9

### Question

Why do companies use Artifactory/Nexus?

### Answer

They can host internal artifacts, proxy/cache external dependencies, control approved dependencies, and provide a centralized repository for team/CI systems.

---

# 80\. Scenario 10

### Question

What is the difference between Maven and Spring Boot?

### Answer

They solve different problems.

```
Maven
 ↓
Build + Dependency Management

Spring Boot
 ↓
Application Framework + Auto-configuration + Bootstrapping
```

Spring Boot projects commonly use Maven as their build tool, but Spring Boot and Maven are not the same thing.

---

# 81\. Common Interview Traps

## Trap 1: "JAR means executable"

❌ Not always.

A JAR can be:

```
Library JAR
```

or:

```
Executable JAR
```

---

## Trap 2: "Maven is a programming language"

❌ No.

Maven is a:

> Build automation and dependency management tool.

---

## Trap 3: "Maven downloads dependencies directly every time"

❌ No.

Maven normally uses the local repository/cache first and retrieves missing artifacts from configured remote repositories.

---

## Trap 4: "`mvn package` only packages"

❌ No.

It runs preceding phases in the same lifecycle first.

---

## Trap 5: "`mvn install` installs the application on the computer"

❌ Not in the usual sense.

It installs the built artifact into the local Maven repository.

```
~/.m2/repository
```

---

## Trap 6: "SNAPSHOT means broken software"

❌ Not necessarily.

It indicates a development version whose contents can change.

---

## Trap 7: "`clean` belongs to the default lifecycle"

❌ No.

Clean is a separate Maven lifecycle.

---

## Trap 8: "Maven and JVM do the same thing"

❌ No.

```
Maven → builds/manages project
JVM   → executes Java bytecode
```

---

# 82\. One-Page Revision Sheet

```
JAVA
 │
 ├── .java
 │      ↓
 │    compile
 │      ↓
 └── .class
        ↓
       JVM

JAR
 │
 ├── .class files
 ├── resources
 └── metadata

MAVEN
 │
 ├── Dependency Management
 ├── Compilation
 ├── Testing
 ├── Packaging
 ├── Installation
 └── Deployment

POM
 │
 ├── groupId
 ├── artifactId
 ├── version
 ├── dependencies
 ├── properties
 ├── plugins
 └── build configuration

REPOSITORIES
 │
 ├── Local
 │     └── ~/.m2/repository
 │
 ├── Maven Central
 │
 └── Private Remote
       ├── Artifactory
       └── Nexus

DEFAULT LIFECYCLE
 │
 ├── validate
 ├── compile
 ├── test
 ├── package
 ├── verify
 ├── install
 └── deploy

OTHER LIFECYCLES
 │
 ├── clean
 └── site

IMPORTANT COMMANDS
 │
 ├── mvn compile
 ├── mvn test
 ├── mvn package
 ├── mvn clean
 ├── mvn clean package
 ├── mvn clean install
 ├── mvn deploy
 ├── mvn dependency:tree
 └── mvn help:effective-pom
```

---

# 83\. 30-Second Interview Explanation of Maven

If an interviewer asks:

> **"Explain Maven."**

A strong answer:

> Maven is a build automation and dependency management tool commonly used in Java applications. We define project information and dependencies in `pom.xml`. Maven resolves direct and transitive dependencies from configured repositories, compiles source code, runs tests, packages applications into artifacts such as JARs, and can install or deploy those artifacts. Maven also provides standard project structure and predefined build lifecycles such as `validate`, `compile`, `test`, `package`, `install`, and `deploy`.

---

# 84\. 60-Second Explanation: What Happens During `mvn clean install`?

A strong interview answer:

> First, Maven executes the clean lifecycle and removes the generated `target` directory. Then it enters the default lifecycle and runs the required phases up to `install`: validation, compilation, testing, packaging, verification, and finally installation. The resulting artifact is placed into the local Maven repository, usually under `~/.m2/repository`, so other Maven projects on the same machine can use it.

---

# 85\. The Most Important Things to Memorize

If you are short on time, memorize these:

### 1\. JAR

```
JAR = Java Archive
```

Package of classes/resources/metadata.

### 2\. Maven

```
Build + Dependency Management
```

### 3\. POM

```
pom.xml = Maven project configuration
```

### 4\. GAV

```
groupId + artifactId + version
```

### 5\. Local Repository

```
~/.m2/repository
```

### 6\. Build Output

```
target/
```

### 7\. Lifecycle

```
validate
→ compile
→ test
→ package
→ verify
→ install
→ deploy
```

### 8\. Clean

Separate lifecycle:

```
mvn clean
```

### 9\. Package vs Install

```
package → creates artifact
install → creates artifact + puts it in local repository
```

### 10\. Install vs Deploy

```
install → local repository
deploy  → remote repository
```

### 11\. Dependency Debugging

```
mvn dependency:tree
```

### 12\. Effective Configuration

```
mvn help:effective-pom
```

---

# 86\. Final Mental Model

The entire lecture can be remembered with this single picture:

```
                         YOUR JAVA PROJECT
                                │
                                ▼
                           pom.xml
                                │
              ┌─────────────────┼─────────────────┐
              │                 │                 │
              ▼                 ▼                 ▼
         Dependencies        Plugins          Metadata
              │                 │                 │
              ▼                 ▼                 ▼
        Maven Repository   Build Lifecycle       GAV
              │                 │
              │          ┌──────┴───────┐
              │          │              │
              │          ▼              ▼
              │       Compile         Test
              │          │              │
              │          └──────┬───────┘
              │                 ▼
              │              Package
              │                 │
              │                 ▼
              │             target/app.jar
              │                 │
              └─────────────────┤
                                ▼
                          Install / Deploy
                                │
                    ┌───────────┴───────────┐
                    ▼                       ▼
             Local .m2 Repository      Remote Repository
```

> **Core idea:** Maven takes your source code + dependency declarations + build configuration and automates the process of turning them into a usable, tested, packaged artifact.

---

# 87\. Quick Self-Test

Try answering these without looking above:

1. What does JAR stand for?
2. Is every JAR executable?
3. What is the difference between a library JAR and an executable JAR?
4. What is the classpath?
5. What is Maven?
6. What is `pom.xml`?
7. What are GAV coordinates?
8. What is a transitive dependency?
9. What is the `.m2` directory?
10. What is the `target` directory?
11. What happens during `mvn compile`?
12. What happens during `mvn test`?
13. What happens during `mvn package`?
14. What does `mvn install` do?
15. What does `mvn deploy` do?
16. Is `clean` part of the default lifecycle?
17. What does `mvn clean install` do?
18. What is Maven Central?
19. Why do companies use Artifactory/Nexus?
20. What is a Parent POM?
21. What is the Super POM?
22. What is the Effective POM?
23. What is a Maven plugin?
24. What is the difference between a lifecycle phase and plugin goal?
25. What does `mvn dependency:tree` help diagnose?
26. What is SNAPSHOT?
27. What is the difference between Maven and JVM?
28. What is the difference between Maven and Gradle?
29. Why are transitive dependencies useful?
30. Why is dependency management important in large projects?
