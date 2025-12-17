# Small gotchas I found through the book

### ⚠️ Spring Bean Creation: `@Component` vs `@Bean`

A common early Spring gotcha:

**Using `@Component` does *not* require a configuration class.**

* If you annotate a class with `@Component` (or `@Service`, `@Repository`, `@Controller`) **and it’s inside a package scanned by Spring Boot**, Spring will automatically detect it and create a bean.
  → **No configuration class needed.**

* You only need a `@Configuration` + `@Bean` when:

  * The class is from a **third-party library** and you cannot annotate it.
  * You want **custom initialization logic**, e.g. constructing the object manually, setting custom params, etc.

**Rule of thumb:**
Use **`@Component`** for your own classes. Use **`@Bean`** only when you must manually construct or customize the bean.

---

### 🔄 Calling `@Bean` Methods Inside a `@Configuration`

When two methods annotated with `@Bean` call each other, Spring treats these calls as requests for **inter-bean references**, *not* normal method calls.

Here’s how it works:

1. If the target bean **already exists in the ApplicationContext**:

   * Spring returns the **existing bean instance**.
   * **It does NOT execute the `@Bean` method again.**

2. If the target bean **does not exist yet**:

   * Spring **creates the bean**, registers it, and returns it.

This mechanism prevents accidental recreation of beans and ensures that each `@Bean` is a **singleton by default**, even if configuration methods call one another.

---

### 🧩 `final` Fields and `@Autowired`

This code will **not compile**:

```java
@Autowired
private final Parrot parrot;
```

Why?

* Spring’s `@Autowired` performs **field injection**, assigning the dependency *after* the object is constructed.
* But a `final` field **must be assigned in the constructor**, so it cannot be left uninitialized for Spring to inject later.

Therefore, Spring cannot assign a dependency to a `final` field via field injection.

✔️ **Correct approaches:**

**1. Use constructor injection (recommended):**

```java
private final Parrot parrot;

@Autowired
public MyClass(Parrot parrot) {
    this.parrot = parrot;
}
```

This works because Spring injects dependencies *before* the object is fully created.

**2. Remove `final` (not recommended):**

```java
@Autowired
private Parrot parrot;
```

Constructor injection + `final` is the safest and most idiomatic Spring approach.

---



### ⚠️ **Constructor Injection vs Setter Injection**

Spring Boot automatically injects dependencies through a class’s **single constructor**, even **without** using `@Autowired`.
If you switch to setter injection, however, Spring **won’t** inject anything unless you explicitly annotate the setter with `@Autowired`.

**Example: Constructor injection (works automatically):**

```java
@Service
public class MyService {
    private final UserRepository repo;

    public MyService(UserRepository repo) {
        this.repo = repo; // injected automatically
    }
}
```

**Example: Setter injection (will NOT work without @Autowired):**

```java
@Service
public class MyService {
    private UserRepository repo;

    // This will NOT be injected unless you add @Autowired
    public void setRepo(UserRepository repo) {
        this.repo = repo;
    }
}
```

👉 **Takeaway:** Prefer **constructor injection**. It’s safer, guarantees required dependencies, works without `@Autowired`, and keeps fields immutable.

---

### ⚠️ **Floating-Point Precision**

If you need to store and operate on decimal values **without losing precision** (e.g., money, measurements, or calculations requiring exact results), **do not use `float` or `double`**.

Floating-point types can introduce rounding errors due to how they’re represented in binary.

✅ **Use `BigDecimal` instead** to preserve decimal accuracy across calculations.

---
**⚠️ `@Transactional` Overrides**

If you annotate **both a class and its methods** with `@Transactional`, be aware that **the method-level annotation takes precedence**.

This means any configuration on the method (e.g. propagation, isolation, read-only, rollback rules) will **override** the class-level settings for that method.

```java
@Transactional(readOnly = true)
public class UserService {

    @Transactional
    public void updateUser(User user) {
        // NOT read-only — method-level config wins
    }
}
```

👉 Use method-level annotations intentionally, as they can silently change transactional behavior defined at the class level.
