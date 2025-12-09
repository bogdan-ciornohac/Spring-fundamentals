<h1 style="font-size: 5rem; font-weight: bold; text-align:center;">
🌱 SPRING FUNDAMENTALS
</h1>

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

Today’s web applications often rely on **dynamic pages**—views that change based on the data sent from the server. A dynamic view receives this variable data from a controller, which prepares the response based on the client’s request.

---

## 🎨 Dynamic Views Using Template Engines

A template engine (e.g., **Thymeleaf**, Mustache, FreeMarker, JSP) helps you easily bind server-side data into HTML pages.
When a controller prepares the response, the template engine renders the HTML using the dynamic data it receives.

---

## 📤 Sending Data from Client to Server

A client can send data to the backend in two main ways:

### ✔️ Request parameters — optional data

Handled in Spring MVC using:

```java
@RequestMapping("/search")
public String search(@RequestParam(required = false) String query) {
    ...
}
```

### ✔️ Path variables — **mandatory** data

Used when the path itself includes the information:

```java
@RequestMapping("/users/{id}")
public String getUser(@PathVariable Long id) {
    ...
}
```

**Regulă generală:**

* Folosește **path variables** pentru valori obligatorii
* Folosește **request parameters** pentru opționale

---

## 🌍 HTTP Methods and `@RequestMapping`

Un request HTTP este identificat prin:

1. Un **path**
2. Un **HTTP method** (GET, POST, PUT, PATCH, DELETE)

Dacă folosești doar `@RequestMapping`, specifici metoda în felul următor:

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

## 🌐 Forms vs HTTP Methods

Browserele, prin formulare HTML, pot trimite doar:

* **GET**
* **POST**

Pentru PUT, PATCH sau DELETE, trebuie folosit JavaScript:

```javascript
fetch('/users/1', { method: 'DELETE' });
```

---

## 🧠 Summary

* Folosești un template engine pentru a crea pagini dinamice
* Controller-ul trimite date către view pentru randare
* Pentru date trimise din browser:

  * `@RequestParam` → opțional
  * `@PathVariable` → obligatoriu
* Acțiunile controllerului se definesc cu `@RequestMapping` + `method = RequestMethod.X`
* HTML forms suportă doar GET și POST — pentru celelalte metode ai nevoie de JavaScript

---
