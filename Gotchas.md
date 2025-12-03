# Small gotchas i found through the book

### ⚠️ Spring Bean Creation: `@Component` vs `@Bean`

A common early Spring gotcha:

**Using `@Component` does *not* require a configuration class.**

* If you annotate a class with `@Component` (or `@Service`, `@Repository`, `@Controller`)
  **and it’s inside a package scanned by Spring Boot**,
  Spring will automatically detect it and create a bean.
  → **No configuration class needed.**

* You only need a `@Configuration` + `@Bean` when:

  * The class is from a **third-party library** and you cannot annotate it.
  * You want **custom initialization logic**, e.g. constructing the object manually, setting custom params, etc.

**Rule of thumb:**
Use **`@Component`** for your own classes.
Use **`@Bean`** only when you must manually construct or customize the bean.

---