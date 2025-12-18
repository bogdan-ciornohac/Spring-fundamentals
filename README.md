<h1 style="font-size: 5rem; font-weight: bold; text-align:center;">
🌱 SPRING FUNDAMENTALS
</h1>
---

# 📚 Table of Contents — Spring Fundamentals

# 📚 Table of Contents — Spring Fundamentals

1. **[Defining Beans in Spring](#defining-beans-in-spring)**
2. **[Wiring Beans in Spring](#-wiring-beans-in-spring)**
3. **[Using Abstraction in Spring](#-using-abstraction-in-spring)**
4. **[Bean Scope and Lifecycle](#-bean-scope-and-lifecycle)**
5. **[Using Aspects with Spring AOP](#-using-aspects-with-spring-aop)**
6. **[Spring Boot and Spring MVC](#-spring-boot-and-spring-mvc)**
7. **[Implementing Web Apps with Spring Boot and Spring MVC](#-implementing-web-apps-with-spring-boot-and-spring-mvc)**
8. **[Using the Spring Web Scopes](#-using-the-spring-web-scopes)**
9. **[Implementing REST Services](#-implementing-rest-services)**
10. **[Consuming REST Endpoints](#-consuming-rest-endpoints)**
11. **[Using Data Sources in Spring Apps](#-using-data-sources-in-spring-apps)**
12. **[Using Transactions in Spring Apps](#-using-transactions-in-spring-apps)**
13. **[Implementing Data Persistence with Spring Data](#-implementing-data-persistence-with-spring-data)**
14. **[Testing Your Spring App](#-testing-your-spring-app)**

---


# Defining Beans in Spring

In Spring, the **first essential concept** to learn is how to register object instances—called **beans**—into the **Spring Application Context**.
You can think of the Application Context as a **container** (or a *bucket*) that Spring uses to store and manage the objects it creates.
**Spring can manage only the objects that you explicitly add to this context.**

---

## 📌 What Is a Bean?

A **bean** is simply an object that Spring manages: it creates it, configures it, injects it where needed, and controls its lifecycle.
To enable this, you register beans in the Application Context using one of several approaches.

---

## ⭐ Ways to Add Beans to the Spring Context

Spring offers **three primary mechanisms** for registering beans:

1. `@Bean` annotation in a configuration class
2. Stereotype annotations (like `@Component`, `@Service`, `@Repository`)
3. Programmatic registration (e.g., `registerBean()` in Spring 5+)

Each approach has different characteristics and ideal use cases.

---

## 1. Using `@Bean` Annotation

This approach involves writing configuration classes annotated with `@Configuration` and creating methods annotated with `@Bean`.

### ✔️ Advantages

* Allows registering **any object**, not just Spring-managed components.
* Supports **multiple beans of the same type**.
* Great for integrating **third-party classes**, libraries, or objects you cannot annotate.

### ❗ Trade-offs

* Requires more code—each bean needs its own factory method.
* Can make configuration verbose if you register many beans manually.

### 📘 Example

```java
@Configuration
public class AppConfig {

    @Bean
    public PaymentService paymentService() {
        return new PaymentService();
    }
}
```

---

## 2. Using Stereotype Annotations

Spring can automatically discover and register beans for classes annotated with:

* `@Component`
* `@Service`
* `@Repository`
* `@Controller` (and `@RestController`)
* or any custom stereotype annotation

These work together with **component scanning**.

### ✔️ Advantages

* Cleaner and more concise.
* You write less boilerplate.
* Ideal for application classes that you own and can annotate.

### ❗ Limitations

* Only works for **your own classes** or source code you can modify.
* Cannot be used for external library classes.

### 📘 Example

```java
@Service
public class OrderService { }
```

---

## 3. Programmatic Registration (`registerBean()`)

Available in **Spring Framework 5+**, the `registerBean()` method allows dynamic or conditional bean registration at runtime.

### ✔️ Advantages

* Useful when bean creation depends on custom logic (conditions, environment, runtime checks).
* Great for frameworks, libraries, or advanced configurations.

### ❗ Limitations

* Less common in standard applications.
* Can complicate configuration if overused.

### 📘 Example

```java
applicationContext.registerBean(CustomService.class, CustomService::new);
```

---

## 🧭 When to Use Which Approach?

| Approach                      | Best For                                 | Pros                | Cons                           |
| ----------------------------- | ---------------------------------------- | ------------------- | ------------------------------ |
| **`@Bean`**                   | Third-party classes, config-heavy setups | Flexible, explicit  | More boilerplate               |
| **Stereotype Annotations**    | Application code you control             | Clean, easy to read | Limited to annotatable classes |
| **Programmatic Registration** | Dynamic or conditional registration      | Full control        | More complex                   |

---

## 📚 Additional Notes

* Understanding bean definition is foundational for dependency injection, configuration, and building scalable Spring applications.
* You will typically mix these approaches depending on the project’s needs.
* Component scanning + annotations is the most common setup in modern Spring Boot applications.

---


# 🌿 Wiring Beans in Spring

In Spring, the **Application Context** is the area in memory where the framework stores and manages all the objects (beans) it controls.
Any object that needs to benefit from Spring’s features—configuration, lifecycle management, dependency injection—must be registered as a **bean** within this context.

When building an application, objects often need to collaborate. One component delegates work to another, forming a network of interactions.
To enable this behavior, we establish **relationships among beans**, which Spring resolves through **Dependency Injection (DI)**.

---

## 🔗 Ways to Wire Beans Together

Spring provides several approaches to connect one bean to another.

---

## 1. **Referencing Other `@Bean` Methods**

From one `@Bean` method, you can call another `@Bean` method directly:

```java
@Bean
public ServiceA serviceA() {
    return new ServiceA(serviceB()); // direct reference
}
```

Spring ensures that even though the method is called directly, **the bean is created only once**, and the existing instance from the context is returned.

✔️ Simple
✔️ Explicit
❗ Only works inside configuration classes

---

## 2. **Using Method Parameters in `@Bean` Methods**

Spring inspects the parameters of a `@Bean` method and injects the required dependencies automatically:

```java
@Bean
public ServiceA serviceA(ServiceB serviceB) {
    return new ServiceA(serviceB);
}
```

Spring finds a bean of type `ServiceB` in the context and provides it as an argument.

✔️ Clean and readable
✔️ Encourages constructor-based dependency injection
❗ Requires matching bean types

---

## 3. **Using `@Autowired`**

The `@Autowired` annotation can be applied in three different ways:

### ✔️ Field Injection (not recommended for real apps)

```java
@Autowired
private ServiceB serviceB;
```

Easy to demo, but **harder to test** and **not recommended for production**.

---

### ✔️ Constructor Injection (recommended)

```java
@Autowired
public ServiceA(ServiceB serviceB) {
    this.serviceB = serviceB;
}
```

This is the **most widely used** approach in modern Spring applications because:

* it enforces immutability
* makes dependencies explicit
* works naturally with frameworks like Spring Boot
* easy to unit test

Spring automatically injects beans matching the constructor’s parameter types.

---

### ✔️ Setter Injection

```java
@Autowired
public void setServiceB(ServiceB serviceB) {
    this.serviceB = serviceB;
}
```

Not widely used today, but can be useful when properties must be optional or mutable.

---

## 🧠 Dependency Injection & IoC

Whenever Spring assigns values or references to fields, constructor parameters, or method parameters, we say that **Dependency Injection (DI)** occurs.

DI is part of the broader principle of **Inversion of Control (IoC)**, where the framework (not your code) controls object creation and wiring.

---

## ⚠️ Circular Dependencies

When two beans depend on each other:

```text
A → B  
B → A
```

Spring **cannot** create them, and the application fails with a circular dependency exception.

To avoid this:

* refactor responsibilities
* introduce interfaces
* redesign the dependency flow

Circular dependencies typically indicate a design issue.

---

## 🆔 Handling Multiple Beans of the Same Type

If Spring finds **more than one bean of the same type**, it cannot determine which one to inject automatically.
You can resolve this using:

### ✔️ `@Primary`

Marks a bean as the default candidate:

```java
@Primary
@Bean
public DataSource mainDataSource() { ... }
```

### ✔️ `@Qualifier`

Specifies exactly which bean you want:

```java
@Autowired
public ReportService(@Qualifier("backupDataSource") DataSource ds) { ... }
```

Useful when different configurations of the same type coexist (e.g., multiple data sources, multiple strategies, etc.).

---

# 🧩 Using Abstraction in Spring

Decoupling your application logic through abstractions is one of the most powerful design practices in software development. By separating **what** a class does (the contract) from **how** it does it (the implementation), you gain flexibility, maintainability, and extensibility in your codebase.

In Java, abstractions are typically created using **interfaces**. Implementations depend on these interfaces instead of each other, reducing coupling and making it easier to modify or replace behaviors without widespread code changes.

---

## 🌱 Why Abstraction Matters

* ✔️ Makes components easier to swap or extend
* ✔️ Reduces dependencies between concrete implementations
* ✔️ Supports cleaner, testable architecture
* ✔️ Allows different implementations to coexist without conflict

Interfaces act as **contracts** between layers or components, ensuring that implementations follow the same behavior while remaining independent from each other.

---

## 🔌 Abstraction + Dependency Injection in Spring

When using dependency injection, Spring understands abstraction naturally:

If a bean depends on an **interface**, Spring searches its context for an instance of a class that implements that interface.

Example (conceptual):

```java
public class PaymentProcessor {
    private final PaymentService paymentService;

    public PaymentProcessor(PaymentService paymentService) {
        this.paymentService = paymentService;
    }
}
```

Spring sees `PaymentService` (the abstraction) and injects a bean that implements it—assuming one is available.

---

## 🏷️ Using Stereotype Annotations

Stereotype annotations (such as `@Component`, `@Service`, `@Repository`) tell Spring to create and register a class as a bean.

Important rules:

* You **only annotate concrete implementations**, never interfaces.
* Spring scans and registers these implementations in the Application Context.
* Spring will later inject them wherever their interface is required.

---

## 👥 Multiple Implementations of the Same Interface

It’s common to have more than one implementation of an abstraction:

```java
public interface NotificationService { ... }

@Service
public class EmailNotificationService implements NotificationService { ... }

@Service
public class SmsNotificationService implements NotificationService { ... }
```

When Spring finds **multiple beans** that satisfy the same abstraction, it needs instructions to choose the correct one.

### To resolve this, you can:

---

### ✔️ 1. Use `@Primary`

Marks one implementation as the default injection candidate:

```java
@Primary
@Service
public class EmailNotificationService implements NotificationService { }
```

Now Spring will inject this implementation unless told otherwise.

---

### ✔️ 2. Use `@Qualifier`

Names a specific bean so you can request it explicitly:

```java
@Service("smsService")
public class SmsNotificationService implements NotificationService { }

@Autowired
public NotificationController(@Qualifier("smsService") NotificationService service) {
    this.service = service;
}
```

This approach is ideal when different implementations are required in different parts of the system.

---

## 🧠 Summary

Abstraction is central to clean architecture, and Spring’s dependency injection system supports it seamlessly:

* Use **interfaces** to decouple implementation details
* Annotate **implementations**, not interfaces
* Spring injects a bean that matches the requested abstraction
* When multiple implementations exist, use:

  * `@Primary` for default selection
  * `@Qualifier` for explicit selection

This practice leads to more flexible, modular, and maintainable Spring applications.

---

# 🔄 Bean Scope and Lifecycle

In Spring, **bean scope** determines *how Spring manages the lifecycle of object instances*. Each scope defines the rules for how many instances of a bean exist and how they are created and consumed throughout the application.

Spring provides two primary scopes for standard applications: **singleton** and **prototype**.

---

## 🟦 Singleton Scope (Default)

A **singleton-scoped** bean means:

* Spring creates **one single instance** of the bean for the entire application context.
* That instance has a **unique bean name**, and whenever you refer to that name, Spring returns the same object.
* By default, Spring instantiates singleton beans **eagerly**, meaning at context startup—unless explicitly configured for lazy initialization.

### 🔥 Lazy vs Eager Initialization

* **Eager (default):** Bean is created as soon as the context loads.
* **Lazy:** Bean is created only when first requested.

### 👥 Thread Safety Concerns

Singleton beans may be accessed by multiple threads simultaneously.
This means:

* They should ideally be **immutable**, or
* You must manage **thread synchronization** manually if mutable state exists.

Singletons are the most commonly used bean type in Spring applications.

---

## 🟧 Prototype Scope

A **prototype-scoped** bean behaves differently:

* Spring associates the scope with the **bean type**, not the instance.
* Spring creates a **new instance of the bean each time it is requested**.
* This is useful for mutable objects or stateful components that should not be shared.

Prototype beans are *not* fully lifecycle-managed.
Spring only handles:

1. Creation
2. Dependency injection

The rest (initialization, destruction, cleanup) becomes your responsibility.

---

## ⚠️ Beware: Injecting Prototype Beans Into Singleton Beans

Injecting a prototype bean into a singleton bean creates a frequent design trap:

```text
Singleton bean A depends on prototype bean B
```

Spring resolves dependencies **once**, at creation time.
So the singleton receives **only one instance** of the prototype bean—and it will reuse that same instance indefinitely.

This defeats the purpose of using a prototype scope, whose whole point is to get **a fresh instance for every request**.

This scenario is usually a **design smell**, unless you deliberately add special mechanisms (like `ObjectFactory`, `Provider`, or lookup methods) to request new prototype instances at runtime.

---

## 🧠 Summary

* **Singleton (default):** One shared instance per context
* **Prototype:** A new instance every time the bean is requested
* Singleton beans should be immutable or thread-safe
* Prototype beans are best for stateful or mutable objects
* Avoid injecting a prototype bean into a singleton without a mechanism to fetch new instances
* Beans can be created eagerly or lazily

---

# 🎯 Using Aspects with Spring AOP

In Spring, **aspects** allow you to intercept method executions and attach behavior before, after, or even *in place of* the method’s logic. This technique—Aspect-Oriented Programming (AOP)—helps you **decouple cross-cutting concerns** from business logic, making your application more modular and maintainable.

Common cross-cutting concerns include:

* Logging
* Security checks
* Transaction management
* Auditing
* Performance monitoring

---

## 💡 Why Use Aspects?

An aspect lets you define logic that executes during a method invocation **without modifying the method itself**.
This keeps business code clean and focused, while support logic lives separately.

However, aspects should be used carefully:

* ✔️ They can simplify complex systems
* ❗ Overuse or misuse makes code **harder to read and debug**
* ✔️ Use them only when they genuinely improve structure or reduce duplication

Spring itself uses AOP internally for major features like **transaction management** and **method-level security**.

---

## 🧩 Defining an Aspect in Spring

To create an aspect:

1. Annotate the class with `@Aspect`
2. Ensure Spring creates a bean of that class (e.g., using `@Component` or a `@Bean` method)

```java
@Aspect
@Component
public class LoggingAspect { ... }
```

Spring must manage this object so it can weave the aspect behavior into method calls.

---

## 🎛️ Pointcuts & Advice

To tell Spring **which methods** should be intercepted, you write **AspectJ pointcut expressions**.

These pointcuts are used inside **advice annotations**, which define *when* the aspect runs.

Spring provides five types of advice:

### ✔️ `@Before`

Runs *before* the method executes.

### ✔️ `@After`

Runs *after* the method completes (success or exception).

### ✔️ `@AfterReturning`

Runs *after* the method returns successfully.

### ✔️ `@AfterThrowing`

Runs *if the method throws an exception*.

### ✔️ `@Around` (most powerful)

Wraps the method execution.
You can:

* run logic before the method
* run logic after the method
* modify the return value
* prevent the method from running
* handle exceptions

Most real-world aspects rely on **`@Around`** because of its flexibility.

---

## 🔗 Multiple Aspects and Ordering

If several aspects apply to the same method:

* All will run
* The order may matter
* You can control the execution priority using `@Order`

```java
@Order(1)
@Aspect
public class FirstAspect { ... }

@Order(2)
@Aspect
public class SecondAspect { ... }
```

Lower numbers run first.

Proper ordering is essential when aspects depend on each other (e.g., security before transactions).

---

## 🧠 Summary

* Aspects allow code to run before, after, or instead of method execution
* Use them to isolate cross-cutting concerns, but avoid overengineering
* Define aspects using `@Aspect` and ensure the class is a Spring bean
* Use AspectJ pointcut expressions and advice annotations to control behavior
* `@Around` is the most flexible advice type
* Use `@Order` to control the order when multiple aspects intercept the same method call

---


# 🌐 Spring Boot and Spring MVC

Modern software increasingly relies on **web applications**, which users access through a browser instead of installing desktop software.
To build such systems effectively, developers must understand:

* how web applications communicate, and
* how to implement the server-side logic that responds to HTTP requests.

Spring provides powerful tools to build backend web applications, and **Spring Boot** simplifies the configuration and setup needed to get started.

---

## 🧩 What Is a Web Application?

A web application has two major parts:

### ✔️ Client Side (Frontend)

* Runs in the user’s browser
* Sends HTTP requests to the backend
* Renders the results returned by the server

### ✔️ Server Side (Backend)

* Processes incoming requests
* Executes business logic
* Stores and retrieves data
* Sends back HTTP responses

---

## 🚀 Spring Boot: Convention Over Configuration

Spring Boot is a Spring ecosystem project designed to eliminate boilerplate configuration.
It follows the **convention-over-configuration** principle:

* Provides **default configurations** for common application needs
* Reduces the amount of setup required
* Lets you focus on implementation instead of framework wiring

### ⭐ Dependency Starters

Spring Boot introduces **dependency starters**—bundled dependency groups with compatible versions that supply a specific capability.

Examples:

* `spring-boot-starter-web` – HTTP server + Spring MVC
* `spring-boot-starter-data-jpa` – JPA & database support
* `spring-boot-starter-security` – security features

Starters make it easy to add functionality without manually configuring dozens of libraries.

---

## 🏗️ Servlet Container

To handle HTTP requests, a Java web application requires a **servlet container**, such as:

* **Tomcat** (default in Spring Boot)
* Jetty
* Undertow

A servlet container translates raw HTTP communication into Java method calls.
This means you don’t need to manually implement low-level HTTP protocol handling.

Spring Boot **autoconfigures** a servlet container for you, so your application runs as a ready-to-use web server.

---

## 🧭 Spring MVC: Managing HTTP Requests

Spring Boot also autoconfigures Spring MVC, a framework built around the **Model–View–Controller** design pattern.

Spring MVC provides:

* Controllers
* Request mappings
* Response handling
* Validation
* Interceptors
* View rendering

Together with Spring Boot autoconfiguration, this enables you to write web use cases with **minimal setup**.

---

## 📄 Minimal Flow: From HTTP Request to Response

Thanks to autoconfiguration, you only need:

1. **A controller class** to handle web requests.
2. **An HTML (or template) file** to render the response.

Spring Boot handles everything else—loading the container, registering components, and wiring Spring MVC.

---

## 🕹️ Writing a Spring MVC Controller

Spring uses annotations to define controllers and their actions.

### ✔️ `@Controller`

Marks a class as a Spring MVC controller.

### ✔️ `@RequestMapping`

Defines the URL and HTTP method (GET, POST, etc.) handled by a controller method.

Example:

```java
@Controller
public class HomeController {

    @RequestMapping("/home")
    public String homePage() {
        return "home.html"; // returned view
    }
}
```

This minimal example shows the core of Spring MVC: mapping a request to a method and returning a response.

---

## 🧠 Summary

* Web apps rely on browser–server communication through HTTP
* Spring Boot simplifies backend development with autoconfiguration and starters
* A servlet container (e.g., Tomcat) handles HTTP processing
* Spring MVC manages request routing and response handling
* Developers only need to write controllers and views for basic flows
* `@Controller` and `@RequestMapping` are central annotations for building web endpoints

---

# 🛠️ Implementing Web Apps with Spring Boot and Spring MVC

Modern web applications rely heavily on **dynamic pages**—views whose content changes depending on the data provided for each request.
A dynamic view receives this variable data from a controller, which prepares the response based on client input.

---

## 🎨 Dynamic Views Using Template Engines

A simple way to build dynamic pages in Spring MVC is by using a **template engine**, such as:

* Thymeleaf (most common in Spring Boot)
* Mustache
* FreeMarker
* JSP

A template engine integrates with Spring MVC and renders HTML pages using the data sent from the controller.

---

## 📤 Sending Data from Client to Server

A client can send data to the backend in two main ways:

### ✔️ Request Parameters — optional data

```java
@RequestMapping("/search")
public String search(@RequestParam(required = false) String query) {
    ...
}
```

### ✔️ Path Variables — mandatory data

```java
@RequestMapping("/users/{id}")
public String getUser(@PathVariable Long id) {
    ...
}
```

**Guideline:**

* Use **path variables** for required values
* Use **request parameters** for optional values

---

## 🌍 HTTP Methods and `@RequestMapping`

Every HTTP request is defined by:

1. A **path**
2. An **HTTP method** (GET, POST, PUT, PATCH, DELETE)

Since you're using classic Spring MVC, methods are declared with `@RequestMapping` + `RequestMethod`:

```java
@RequestMapping(value = "/users", method = RequestMethod.GET)
public String listUsers() { ... }

@RequestMapping(value = "/users", method = RequestMethod.POST)
public String createUser() { ... }

@RequestMapping(value = "/users/{id}", method = RequestMethod.PUT)
public String updateUser(@PathVariable Long id) { ... }

@RequestMapping(value = "/users/{id}", method = RequestMethod.DELETE)
public String deleteUser(@PathVariable Long id) { ... }
```

---

## 🌐 HTML Forms and HTTP Methods

Browsers, through regular HTML forms, can submit only:

* **GET**
* **POST**

To perform **PUT**, **PATCH**, or **DELETE**, you must trigger the request using JavaScript:

```javascript
fetch('/users/1', { method: 'DELETE' });
```

---

## 🧠 Summary 

* Template engines allow Spring MVC apps to render dynamic HTML pages
* Controllers provide data to views for dynamic rendering
* Client data can be received via:

  * `@RequestParam` → optional data
  * `@PathVariable` → required data
* Controller actions are mapped using `@RequestMapping` with `RequestMethod.X`
* HTML forms support only GET and POST; JavaScript is required for other HTTP methods

---
# 🌐 Using the Spring Web Scopes

Beyond the standard **singleton** and **prototype** scopes (discussed in earlier chapters), Spring provides three additional scopes that apply **exclusively to web applications**. These scopes help you manage bean lifecycles tied to HTTP requests and sessions.

These are known as **Spring Web Scopes**:

---

## 🟦 1. Request Scope

**Definition:**
A new instance of the bean is created **for each HTTP request**.

**Characteristics:**

* One instance *per request*
* Automatically cleaned up when the request finishes
* Ideal for storing request-specific data
* No concurrency concerns—each bean instance is accessed by a single request only

**Design considerations:**

* Avoid heavy initialization in constructors or `@PostConstruct`
  → Request-scoped beans are created very often.

---

## 🟧 2. Session Scope

**Definition:**
One bean instance is created **per HTTP session**, meaning multiple requests from the same user can share the same instance.

**Characteristics:**

* Shared across requests from the same client session
* Useful for storing data that persists across multiple interactions
* Not shared between different users

**Concurrency warning:**
Even if the requests come from the same user, the browser may send **parallel requests**.
If these modify the session-scoped bean, **race conditions** can occur.

You must either:

* avoid concurrent writes, or
* synchronize access to mutable state

---

## 🟥 3. Application Scope

**Definition:**
A single bean instance exists **for the entire lifetime of the web application**, shared across all clients and all requests.

**Characteristics:**

* Global to the whole application
* Never garbage-collected while the app runs
* Every request from any client accesses the same instance

**Recommendation:**
Avoid application-scoped beans.

Why?

* Any write requires synchronization → performance bottlenecks
* Long-lived objects → increased memory pressure
* Encourages a stateful architecture → harder to scale

A better approach is to store shared data in a **database**, not in application-scoped beans.

---

## ⚠️ Stateful Applications & Architectural Concerns

Both session-scoped and application-scoped beans introduce **shared state**, meaning:

* Requests become less independent
* The application becomes **stateful**

Stateful applications can suffer from:

* Concurrency issues
* Scalability limitations
* Complex debugging and deployment behavior

While these topics go beyond the scope of this chapter, it is important to understand that **minimizing shared state leads to more resilient and scalable systems**.

---

## 🧠 Summary

| Scope           | Lifecycle         | Shared By                        | Recommended Usage                                 |
| --------------- | ----------------- | -------------------------------- | ------------------------------------------------- |
| **Request**     | Per HTTP request  | Nobody (isolated)                | Request-specific data                             |
| **Session**     | Per user session  | Multiple requests from same user | User interaction state (careful with concurrency) |
| **Application** | Whole application | All clients                      | Generally avoid; store shared data in DB instead  |

---

# 🌐 Implementing REST Services

Representational State Transfer (**REST**) is a simple and widely adopted architectural style for enabling communication between different applications. Spring provides first-class support for implementing RESTful web services through **Spring MVC**.

This chapter describes how to create REST endpoints, return HTTP responses, structure REST controllers, and handle exceptions effectively.

---

## 🚀 Building REST Endpoints with Spring MVC

To expose a REST endpoint, Spring must know that your controller’s methods return **data**, not HTML views. There are two ways to inform Spring of this:

### ✔️ Option 1: Use `@ResponseBody` on the method

```java
@Controller
public class HelloController {

    @RequestMapping("/hello")
    @ResponseBody
    public String hello() {
        return "Hello World!";
    }
}
```

`@ResponseBody` tells Spring to write the return value directly into the HTTP response body.

---

### ✔️ Option 2: Use `@RestController` on the class

```java
@RestController
public class HelloController {

    @RequestMapping("/hello")
    public String hello() {
        return "Hello World!";
    }
}
```

`@RestController` is a convenience annotation that combines:

* `@Controller`
* `@ResponseBody`

Using neither of these tells Spring to treat the return value as a **view name**, causing the dispatcher servlet to search for a view template.

---

## 📤 Returning HTTP Responses

### ✔️ Relying on Spring's default behavior

A controller method can return the response body directly:

```java
@RequestMapping("/greet")
public String greet() {
    return "Hi!";
}
```

Spring selects the appropriate HTTP status code automatically.

---

### ✔️ Customizing the response: `ResponseEntity`

If you need full control over:

* HTTP status code
* Headers
* Response body

you can return a `ResponseEntity`:

```java
@RequestMapping("/status")
public ResponseEntity<String> customStatus() {
    return ResponseEntity
        .status(HttpStatus.CREATED)
        .header("X-Custom-Header", "Test")
        .body("Resource created");
}
```

---

## ⚠️ Handling Exceptions

### ✔️ Approach 1: Handle exceptions inside the controller method

This keeps error handling close to the logic but may duplicate code across methods.

### ✔️ Approach 2: Use a REST controller advice

A **global exception handler** keeps error handling clean and reusable.

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(NotFoundException.class)
    public ResponseEntity<String> handleNotFound(NotFoundException ex) {
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(ex.getMessage());
    }
}
```

This approach decouples exception processing from controller logic and avoids repetition.

---

## 📥 Getting Data From the Client

A REST endpoint can receive data through:

* **Request parameters**

  ```java
  @RequestParam String filter
  ```

* **Path variables**

  ```java
  @PathVariable Long id
  ```

* **Request body**

  ```java
  @RequestBody UserDto user
  ```

Each mechanism serves different use cases depending on whether data is optional, required, or complex.

---

## 🧠 Summary

* REST enables simple communication between applications
* Use `@ResponseBody` or `@RestController` to build REST endpoints
* Spring can return the HTTP response body directly or through `ResponseEntity`
* Exceptions can be handled locally or globally using a controller advice
* Clients can send data via request parameters, path variables, or the request body

---

# 🌐 Consuming REST Endpoints

In real-world backend architectures, it is common for one backend application to communicate with another by calling its exposed REST endpoints. Spring provides multiple tools to help you implement the **client side** of a REST service.

This chapter introduces the most relevant solutions and helps you understand when to use each one.

---

## 🔌 Options for Calling REST Endpoints in Spring

Spring offers several mechanisms for consuming REST APIs. The three most important ones today are:

---

## 🟦 1. OpenFeign (Spring Cloud)

**OpenFeign** is a declarative HTTP client provided through the **Spring Cloud** ecosystem.

### ✔️ Key features:

* Minimal boilerplate—define an interface, and Feign generates the REST client
* Built-in support for:

  * Load balancing
  * Resilience (when combined with Spring Cloud components)
  * Request/response logging
  * Interceptors
* Ideal for microservices and distributed systems

Feign significantly simplifies the code needed to consume REST endpoints.

---

## 🟧 2. RestTemplate (Legacy)

**RestTemplate** has been the traditional way to call REST endpoints in Spring applications.

However:

> ❗ You should **not use RestTemplate in new applications**.
> It is in maintenance mode and will not receive major new features.

Existing codebases may still use it, but modern implementations should prefer Feign or WebClient.

---

## 🟩 3. WebClient (Reactive)

**WebClient** is Spring’s modern HTTP client built for **reactive applications**.

### ✔️ Benefits:

* Fully non-blocking
* Supports functional reactive programming with Project Reactor
* High performance for concurrent or streaming workloads

### ❗ Important:

WebClient is excellent, but **only for applications designed around a reactive programming model**.
Using it in a nonreactive application adds unnecessary complexity.

---

## 🎯 Choosing the Right Tool

Choosing between Feign and WebClient depends on the architecture of your app:

### ✔️ For standard, nonreactive Spring Boot applications

**Use OpenFeign.**
It offers the cleanest API, the easiest configuration, and strong ecosystem support.

### ✔️ For reactive applications

**Use WebClient.**
It takes full advantage of non-blocking IO and reactive pipelines.

### ❌ Avoid RestTemplate for new work

Although simple, it is no longer the preferred approach in modern Spring development.

---

## 🧠 Summary

* Backend services often need to consume REST endpoints exposed by other services
* Spring provides several HTTP clients: OpenFeign, RestTemplate, and WebClient
* **OpenFeign** → best choice for standard (blocking) applications
* **WebClient** → best choice for reactive applications
* **RestTemplate** → legacy; avoid using in new implementations

---

# 💾 Using Data Sources in Spring Apps

Most real-world applications need to store and retrieve data from a relational database. In Java, the Java Development Kit (JDK) provides the foundational **JDBC abstractions** required to interact with databases. However, your application also needs a **JDBC driver**—a runtime dependency that provides the actual implementation for connecting to a specific database (MySQL, PostgreSQL, Oracle, etc.).

To manage database connections efficiently, Spring Boot and Spring Framework offer tools that simplify and optimize JDBC usage.

---

## 🏗️ What Is a Data Source?

A **data source** is an object that manages the creation and pooling of connections to a database server.

Without a data source:

* Each operation would open a new database connection
* Opening connections repeatedly is slow
* Performance would degrade quickly

A data source ensures your application uses a **connection pool**, reusing existing connections instead of creating new ones.

---

## ⚡ HikariCP — Spring Boot’s Default Data Source

Spring Boot automatically configures a high-performance connection pool called **HikariCP**.

### Benefits of HikariCP:

* Efficient connection reuse
* Reliable performance under high load
* Minimal configuration required

You can replace HikariCP with another implementation (e.g., Apache DBCP, Tomcat JDBC Pool) if your application has special requirements.

---

## 🛠️ Working with JdbcTemplate

`JdbcTemplate` is a Spring-provided utility that simplifies JDBC operations.

It depends on a `DataSource` to establish connections and allows you to:

### ✔️ Update data

Use the `update()` method for INSERT, UPDATE, or DELETE statements:

```java
jdbcTemplate.update("INSERT INTO users(name) VALUES(?)", "Alice");
```

### ✔️ Query data

Use one of the `query()` methods to run SELECT queries and map results:

```java
List<User> users = jdbcTemplate.query(
    "SELECT * FROM users",
    (rs, rowNum) -> new User(rs.getLong("id"), rs.getString("name"))
);
```

`JdbcTemplate` removes boilerplate code like manually opening/closing connections, handling exceptions, and iterating through result sets.

---

## 🔧 Customizing the Data Source and JdbcTemplate

Spring Boot auto-configures the default `DataSource` and `JdbcTemplate`.
However, you can override these defaults by declaring your own beans:

### Custom Data Source

```java
@Bean
public DataSource customDataSource() {
    // custom configuration
}
```

If Spring finds a custom `DataSource` bean, it uses it instead of the default HikariCP configuration.

### Custom JdbcTemplate

```java
@Bean
public JdbcTemplate customJdbcTemplate(DataSource dataSource) {
    return new JdbcTemplate(dataSource);
}
```

Custom configurations are useful when:

* Connecting to nonstandard databases
* Needing performance tuning
* Using specialized connection pool settings

---

## 🗄️ Using Multiple Data Sources

If your application needs to connect to **more than one database**, you can:

1. Define multiple `DataSource` beans
2. Define one `JdbcTemplate` per data source
3. Use `@Qualifier` to disambiguate

Example:

```java
@Autowired
public JdbcTemplate(
    @Qualifier("ordersDataSource") DataSource ds
) { ... }
```

This follows the same pattern described in earlier chapters about handling multiple beans of the same type.

---

## 🧠 Summary

* JDBC requires a **JDBC driver** to connect to a relational database
* A **data source** manages database connections and improves performance via pooling
* Spring Boot uses **HikariCP** by default but allows full customization
* **JdbcTemplate** simplifies database operations for both updates and queries
* You can override default configurations by defining your own `DataSource` or `JdbcTemplate` beans
* Multiple data sources require qualifiers to distinguish them

---

# 🔒 Using Transactions in Spring Apps

A **transaction** is a group of data-changing operations that must either **all succeed** or **all fail together**. This all-or-nothing behavior ensures data consistency, even when unexpected errors occur. In real-world applications, almost every use case that modifies persistent data should run inside a transaction.

---

## 🧩 What Happens in a Transaction?

### ✔️ Commit

If **all operations** succeed, the transaction **commits**.
This means the application permanently saves the changes made during the use case.

### ✔️ Rollback

If **any operation fails**, Spring restores the database to its original state from before the transaction started.
This is called a **rollback**.

Rollback protects your application from partial updates, inconsistent states, and corrupted data.

---

## 🛠️ Using `@Transactional` in Spring

Spring makes implementing transactional behavior extremely simple through the `@Transactional` annotation.

You can apply `@Transactional` to:

### ✔️ A Method

Marks that specific method to run inside a transaction.

```java
@Transactional
public void processOrder() {
    // all operations here are part of the same transaction
}
```

### ✔️ A Class

All methods in the class become transactional.

```java
@Transactional
public class OrderService {
    ...
}
```

Annotating the class is useful when **all operations** in the service are expected to participate in transactions.

---

## ⚙️ How Spring Implements Transactions

When your application runs, **Spring AOP** handles the transactional behavior behind the scenes.

Here’s what happens:

1. A Spring **aspect** intercepts calls to methods annotated with `@Transactional`.
2. The aspect **starts a new transaction** before the method runs.
3. If the method **throws an exception**, the aspect **rolls back** the transaction.
4. If the method completes **without throwing an exception**, the aspect **commits** the transaction.

This mechanism lets you focus on writing business logic without manually managing database transactions.

---

## 🧠 Summary

* A transaction groups data-changing operations into a single atomic unit
* Spring commits the transaction if everything succeeds
* Spring rolls back the transaction if any exception occurs
* Use `@Transactional` on methods or classes to mark them transactional
* Spring uses AOP aspects to start, commit, and roll back transactions automatically

---

# 🗄️ Implementing Data Persistence with Spring Data

**Spring Data** is a powerful project in the Spring ecosystem that simplifies the implementation of the persistence layer in Spring applications. It provides a unified abstraction over various persistence technologies and offers a consistent set of repository contracts, reducing boilerplate and improving maintainability.

---

## 🌱 Spring Data Repository Abstractions

Spring Data allows you to implement repositories simply by defining **interfaces** that extend predefined contracts. These contracts determine the operations your repository can perform.

### ✔️ `Repository`

* The base marker interface
* Does **not** provide CRUD operations
* Used mainly to define custom sets of behavior

### ✔️ `CrudRepository`

* Extends `Repository`
* Provides standard **CREATE, READ, UPDATE, DELETE (CRUD)** operations

### ✔️ `PagingAndSortingRepository`

* Extends `CrudRepository`
* Adds built-in support for:

  * Pagination
  * Sorting

These abstractions eliminate the need to write repetitive persistence logic.

---

## 🧩 Choosing the Correct Spring Data Module

Spring Data supports multiple persistence technologies through specialized modules.
Choose the module that matches your database technology:

* **Spring Data JDBC** → relational databases via JDBC
* **Spring Data JPA** → relational databases using JPA/Hibernate
* **Spring Data MongoDB** → MongoDB
* **Spring Data Redis** → Redis
* Others exist for Cassandra, Neo4j, Elasticsearch, and more

Each module implements the repository contracts in a way that fits the underlying technology.

---

## 🛠️ Defining Custom Repository Methods

When you extend a Spring Data repository contract, your repository automatically inherits the provided operations.
You can also define **custom methods** in the interface.

There are **two** ways to implement custom queries:

---

## 1️⃣ Using `@Query` Annotation (Recommended)

```java
@Query("SELECT * FROM users WHERE email = :email")
User findByEmail(String email);
```

Advantages:

* Clear, explicit SQL or JPQL
* Easy to maintain
* No method name translation required
* Less risk during refactoring

This is the **preferred** approach for custom operations.

---

## 2️⃣ Relying on Method Name Query Derivation (Not Recommended)

Spring Data can translate certain method names into SQL/JPQL queries:

```java
User findByEmailAndStatus(String email, Status status);
```

However, this approach has drawbacks:

* Long and unreadable method names for complex queries
* Slower application startup due to method name parsing
* Requires learning naming conventions
* Risky during refactoring—renaming a method can cause runtime failures

If Spring Data cannot interpret the method name, the application **fails to start**.

Because of these issues, it's generally better to use `@Query`.

---

## ⚠️ Modifying Operations Require `@Modifying`

Any operation that **changes data**—such as INSERT, UPDATE, or DELETE—must be annotated with:

```java
@Modifying
@Query("UPDATE users SET active = false WHERE id = :id")
void deactivateUser(Long id);
```

`@Modifying` tells Spring Data that the method performs a write operation rather than a SELECT query.

---

## 🧠 Summary

* Spring Data simplifies persistence through repository abstractions
* Extend `Repository`, `CrudRepository`, or `PagingAndSortingRepository` depending on your needs
* Choose the Spring Data module based on your database technology
* Use `@Query` to clearly define custom queries
* Avoid relying on method name query derivation for complex operations
* Use `@Modifying` for any repository method that changes data

---

# 🧪 Testing Your Spring App

Testing is an essential part of software development. A **test** is a small piece of code written to validate the behavior of a particular part of your application. Tests help ensure that new changes do not break existing functionality and act as living documentation for how your code is intended to behave.

Spring applications support multiple testing approaches, and understanding when and how to test different parts of your system is fundamental to building reliable software.

---

## 🧩 Types of Tests

In general, tests fall into **two major categories**:

---

## 🟦 1. Unit Tests

A **unit test** focuses on a *single, isolated* component—usually a method or a small class.

### ✔️ Characteristics:

* Validates how one piece of logic behaves in isolation
* Runs very fast
* Helps identify problems in a small component directly
* Does not load the entire Spring context

Unit tests are ideal for testing pure logic or business rules without involving databases, controllers, or external systems.

---

## 🟧 2. Integration Tests

An **integration test** validates how two or more components work *together*.

### ✔️ Characteristics:

* Ensures multiple components integrate correctly
* May involve loading the Spring context
* Often interacts with real infrastructure (database, controllers, etc.)
* Slower than unit tests but essential for full correctness

Integration tests catch issues that don’t show up when testing components individually.

---

## 🧪 Using Mocks in Testing

Sometimes, you want your test to focus only on a specific set of interactions and ignore others.
In these cases, you replace real components with **mocks**—fake objects under your control.

### ✔️ Why use mocks?

* Remove dependencies that you do not want to test
* Allow precise control over method responses
* Help focus tests on the behavior of one or a few components
* Enable predictable and repeatable test outcomes

Mocking tools (such as Mockito) are commonly used with Spring tests.

---

## 🔄 Structure of a Good Test

Every test generally follows **three main steps**:

### 1️⃣ Assumptions (Arrange)

* Define input values
* Configure mock behavior
* Prepare the environment for testing

### 2️⃣ Execution (Act)

* Call the method or component you want to test

### 3️⃣ Validations (Assert)

* Verify that the method behaved as expected
* Check returned values, interactions, or side effects

This pattern keeps tests clean, readable, and easy to maintain.

---

## 🧠 Summary

* Tests ensure stable development and act as documentation
* **Unit tests** validate isolated logic and run quickly
* **Integration tests** validate how components work together
* Use **mocks** to remove unwanted dependencies during tests
* A well-structured test follows: **Arrange → Act → Assert**

