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
