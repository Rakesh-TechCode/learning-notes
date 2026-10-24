# Spring Core — Topic 1: IoC, DI & Beans

## 1. What problem does Spring Framework solve?

**Q: Why do we use Spring?**

Creating dependencies manually using `new` tightly couples classes and makes dependency replacement and testing harder. Spring provides IoC and DI to manage object creation, inject dependencies, and manage bean lifecycle callbacks.

**Remember:** Spring manages Spring-created beans; objects created manually with `new` are not automatically managed by Spring.

## 2. IoC vs DI

- **IoC (Inversion of Control):** Spring controls object creation and management instead of the application creating and managing every dependency itself.
- **DI (Dependency Injection):** Spring supplies the dependencies a class requires.

**Memory trick:** IoC = who controls object creation? DI = how does the object receive its dependencies?

## 3. How does Spring create and manage objects?

**Preserve this interview answer:**

Spring scans for stereotype annotations, registers bean definitions in the application context, creates the required beans, and injects their dependencies through constructors. By default, Spring beans use singleton scope, so the same managed instance is reused within that bean definition and container.

**Flow:** Component scanning → Bean definitions → Bean creation → Dependency injection → Initialization → Ready for use.

## 4. Constructor vs Setter vs Field Injection

| Constructor injection | Setter injection | Field injection |
|---|---|---|
| Dependency supplied through constructor | Dependency supplied through setter | Dependency injected directly into field |
| Required dependencies | Useful for optional dependencies | Less explicit |
| Supports `final` fields | Assigned through a setter, so field generally isn't `final` | Cannot use `final` field for normal field injection |
| Easy unit testing without Spring | Allows setter-based reassignment | Harder to test without reflection or Spring |

**Why prefer constructor injection?**
- Makes required dependencies explicit.
- Supports `final` fields.
- Makes unit testing easier by passing mocks directly.
- Helps prevent objects from being created without required dependencies.

A single constructor does not need `@Autowired` in modern Spring.

### Special note: Setter vs Constructor

**When should you use each?**

**Constructor injection**
- For required dependencies
- Dependency is supplied during construction
- Can store dependency in a `final` field
- Usually preferred

**Setter injection**
- For optional dependencies
- Dependency can be supplied or changed afterward
- Field generally cannot be `final` if assigned in the setter
- Useful when a dependency is genuinely optional or needs setter-based configuration

**Preserve these distinctions:**
- Constructor injection: A `final` field can prevent reassignment after construction.
- Setter injection: A non-`final` field can be reassigned through the setter.
- Spring does not automatically replace the dependency every time a business method is called; the setter must actually be called to change the reference.

**Exact interview answer:** “Setter injection allows a dependency to be reassigned through a setter method after object construction, whereas constructor injection commonly uses `final` fields for required dependencies.”

## 5. Stereotype annotations

| Annotation | Typical purpose |
|---|---|
| `@Component` | General-purpose Spring-managed component |
| `@Service` | Business logic |
| `@Repository` | Data access layer |
| `@Controller` | Spring MVC web controller |

All are component-scanning candidates. `@RestController` combines `@Controller` and `@ResponseBody`.

A class isn't automatically registered just because another class injects it. It must be registered through component scanning, a `@Bean` method, or another supported configuration mechanism.

## 6. `@Component` vs `@Bean`

- `@Component`: Put it on a class so Spring can discover it through component scanning.
- `@Bean`: Put it on a method; Spring registers the object returned by that method.

**Memory trick:** `@Component` → Spring discovers the class. `@Bean` → Spring registers the returned object.

**Third-party class:** Use a `@Configuration` class with a `@Bean` method when you cannot annotate the third-party class.

## 7. `@Component` vs `@Configuration`

**Main difference:**

| `@Component` | `@Configuration` |
|---|---|
| Registers a class through component scanning. | Marks a class as a source of bean definitions. |
| Can also contain `@Bean` methods. | Commonly contains `@Bean` methods. |
| Calls between its `@Bean` methods aren't intercepted to ensure the managed bean is reused. | In the default full configuration mode, calls between `@Bean` methods are intercepted to reuse Spring-managed beans. |

### When to use each?

- `@Component` — Use for a regular class that Spring should discover and manage, such as a custom helper class.
- `@Configuration` — Use when defining bean configuration, especially when multiple `@Bean` methods may depend on each other.
- Third-party class? Use `@Configuration` with `@Bean` when you cannot add Spring annotations to that class.

**Remember:** `@Component` and `@Configuration` can both contain `@Bean` methods, but `@Configuration` provides full configuration-class behavior by default.

### Saved interview example

```java
@Configuration
class AppConfig {

    @Bean
    PaymentService paymentService() {
        return new PaymentService();
    }

    @Bean
    OrderService orderService() {
        return new OrderService(paymentService());
    }
}
```

**Interviewer asks:** Why do we use `@Bean` here instead of `@Component`, and what happens when `orderService()` calls `paymentService()`?

**Why `@Configuration` + `@Bean`?** To explicitly define which objects Spring should register as beans, rather than relying on component scanning to discover annotated classes.

**Shared bean assurance:** In default full configuration mode, when `orderService()` calls `paymentService()`, Spring intercepts the call and returns the existing managed `PaymentService` bean instead of creating another instance.

Remember, `@Component` is also a way to register beans. The key difference is how Spring handles calls between `@Bean` methods.

**Exact interview answer:** “`@Configuration` with `@Bean` allows us to explicitly define Spring beans. In full configuration mode, Spring intercepts calls between `@Bean` methods to reuse the managed bean instead of creating duplicate instances.”

**Tricky variation:** With a plain `@Component` containing `@Bean` methods, direct calls between those methods aren't intercepted by default. Each direct call can create a new object.

## 8. Bean lifecycle

**Memory trick:** Create → Inject → Initialize → Use → Shutdown → Destroy.

1. Instantiation: constructor executes.
2. Dependency injection: required dependencies are supplied.
3. Initialization: initialization callbacks execute.
4. Ready for use.
5. Application shutdown.
6. Destruction callbacks execute when Spring destroys the bean.

### `@PostConstruct`

Runs after dependency injection during initialization.

```java
@Service
class PaymentService {

    PaymentService() {
        System.out.println("Constructor");
    }

    @PostConstruct
    void init() {
        System.out.println("PostConstruct");
    }
}
```

Output:
```text
Constructor
PostConstruct
```

**Remember:** `@PostConstruct` executes after dependency injection, not merely after constructor execution.

**Dependency availability:** If a required repository is successfully injected through the constructor, `repository != null` inside `@PostConstruct` prints `true`.

For a singleton bean, `@PostConstruct` runs once during initialization, not on every business-method call.

### `@PreDestroy`

Runs when Spring destroys the bean, typically during graceful application shutdown. It can be used to release resources and perform cleanup.

**Tricky variation:** A forced JVM termination or crash may prevent `@PreDestroy` from executing.

### If `@PostConstruct` throws an exception

Bean initialization fails. If the bean is an eagerly initialized singleton required during startup, the application context typically fails to start.

**What if the bean is lazy?** `@Lazy` delays creation; it does not skip initialization or ignore exceptions. If `@PostConstruct` fails, the requested bean cannot be initialized and used normally.

### `BeanPostProcessor`

An infrastructure interface that lets Spring run custom logic before and after bean initialization.

- `postProcessBeforeInitialization()` runs before initialization callbacks such as `@PostConstruct`.
- `postProcessAfterInitialization()` runs after them.
- The after-initialization method can return a proxy instead of the original bean, supporting proxy-based features such as `@Transactional` and `@Async`.

**Order to remember:** Before initialization → `@PostConstruct` → After initialization.

## 9. Singleton vs Prototype scope

| Singleton | Prototype |
|---|---|
| One instance per bean definition per Spring IoC container | New instance each time the bean is requested from the container |
| Default scope | Must be explicitly configured |
| Destruction callbacks are managed by Spring | Spring doesn't automatically invoke destruction callbacks |

**Exact interview answer:** “Singleton scope provides one instance per bean definition per container, while prototype scope creates a new instance each time the bean is requested from the container. However, constructor-injecting a prototype bean into a singleton does not automatically create a new instance on every method call.”

### Tricky example: Prototype injected into singleton

```java
@Service
@Scope("prototype")
class OrderService {}

@RestController
class OrderController {
    private final OrderService orderService;

    OrderController(OrderService orderService) {
        this.orderService = orderService;
    }

    void execute() {
        orderService.process();
    }
}
```

**What happens?** Spring creates the singleton `OrderController` and requests one prototype `OrderService` while constructing it. The controller retains that reference, so repeated calls to `execute()` use the same instance. A method call on the injected service does not itself request a new bean from Spring.

### How to request a fresh prototype instance

```java
@Autowired
private ObjectProvider<OrderService> orderServiceProvider;

void execute() {
    OrderService service = orderServiceProvider.getObject();
    service.process();
}
```

Each call to `getObject()` requests a prototype bean from Spring, creating a new instance for that request.

**Comparison:**
- Calling a method on an already-injected service → no new prototype instance.
- Calling `getObject()` on `ObjectProvider` again → a new prototype instance.

**Prototype destruction:** Spring manages prototype creation and initialization but does not automatically invoke destruction callbacks. The application must handle cleanup when necessary.

## 10. Circular dependencies

A circular dependency occurs when two or more beans depend on each other, creating a cycle.

- Constructor-injection cycles generally prevent Spring from constructing either bean first and cause startup failure.
- Spring may resolve some cycles involving field or setter injection, but not every cycle.
- `@Lazy` delays dependency resolution using a proxy and can help break an immediate constructor-based cycle.
- Best practice: redesign the service responsibilities to eliminate the circular dependency.

**Interview answer:** “A circular dependency occurs when two or more beans depend on each other, creating a cycle. With constructor injection, Spring cannot create either bean first, so application startup fails.”

**Production fix:** “I would first refactor the service responsibilities to eliminate the circular dependency. I would consider `@Lazy` only when there is a justified reason to defer dependency resolution, not as a default workaround.”

## 11. `@Primary` vs `@Qualifier`

- `@Primary`: Defines the preferred bean when multiple candidates exist.
- `@Qualifier`: Explicitly selects a particular bean for an injection point.

Example:

```java
@Primary
@Service
class UpiPayment implements PaymentMethod {}
```

Or:

```java
PaymentService(@Qualifier("cardPayment")
               PaymentMethod paymentMethod) {
}
```

**Exact interview answer:** “When multiple beans implement the same interface, Spring throws `NoUniqueBeanDefinitionException` if it cannot determine which one to inject. I can use `@Primary` to define a default bean or `@Qualifier` to explicitly select the required implementation.”

**Remember:** `@Qualifier` is the more explicit choice at a particular injection point. `@Primary` supplies the default preference.

## 12. Rapid-fire tricky variations

1. **Is a class automatically a bean because DI is used?** No. It must be registered with Spring.
2. **Does one singleton mean one object across the entire JVM?** No. It means one instance per bean definition per container.
3. **Does a prototype bean injected into a singleton create a new instance per method call?** No.
4. **Does `@PostConstruct` execute after every request?** No, not for an already initialized singleton.
5. **Does `@PreDestroy` execute after every business-method call?** No.
6. **Does `@Lazy` prevent initialization exceptions?** No. It delays bean creation.
7. **Can `@Component` contain `@Bean` methods?** Yes, but direct calls between its methods aren't intercepted in the same way as full `@Configuration`.
8. **Can Spring inject two implementations of the same interface without resolving ambiguity?** Not when both are candidates and no selection rule determines the bean.
9. **Can Spring automatically manage an object created using `new` inside arbitrary business code?** No, not unless it is explicitly registered or managed through an appropriate mechanism.
10. **Does Spring automatically invoke destruction callbacks for prototype beans?** No.

## Final revision checklist

- [ ] Explain Spring, IoC and DI.
- [ ] Explain bean registration and object creation.
- [ ] Compare constructor, setter and field injection.
- [ ] Know stereotypes, `@Component` and `@Bean`.
- [ ] Explain `@Configuration` full-mode method interception.
- [ ] Explain `@PostConstruct`, `@PreDestroy` and `BeanPostProcessor`.
- [ ] Explain singleton/prototype scope and `ObjectProvider`.
- [ ] Explain circular dependencies and practical fixes.
- [ ] Explain `@Primary`, `@Qualifier` and bean ambiguity.

# Spring Boot Fundamentals — Topic 2 Quick-Recall Notes

**Level:** Java Backend Developer — 3+ YOE  
**Status:** ✅ Completed — 12/12 questions

## Q1. Spring Framework vs Spring Boot

**Interview answer:**

Spring Framework provides core features like Dependency Injection, while Spring Boot is built on top of it to reduce manual configuration. It provides auto-configuration, starter dependencies, and embedded servers, making it faster and easier to develop and run Spring applications.

## Q2. Why do we use Spring Boot?

- Reduces manual configuration.
- Provides starter dependencies.
- Configures suitable components automatically.
- Supports embedded servers and executable JARs.
- Simplifies application development and deployment.

**Remember:** Spring Boot reduces configuration; it doesn't eliminate all configuration.

## Q3. What is auto-configuration?

**Interview answer:**

Spring Boot auto-configuration reduces manual setup by checking the libraries available on the classpath and existing beans. Based on these conditions, it automatically configures suitable components. For example, with the web starter, it configures the web infrastructure and embedded Tomcat when the relevant conditions are met.

**Memory trick:**

Dependencies available → conditions checked → suitable configuration applied.

## Q4. What does `@SpringBootApplication` do?

It combines three annotations:

- `@SpringBootConfiguration` — identifies the class as a source of Spring Boot configuration.
- `@EnableAutoConfiguration` — enables Spring Boot auto-configuration.
- `@ComponentScan` — scans the package containing the main class and its subpackages by default.

**Interview trap:** A class outside the scanned package may not be discovered unless scanning is configured appropriately.

## Q5. What are Spring Boot starters?

Starters are convenient dependency bundles for specific purposes.

Example:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

This brings in the commonly needed libraries for REST APIs, including Spring MVC and an embedded server.

**Know the difference:**

- **Starter dependency:** Supplies the required libraries.
- **Auto-configuration:** Configures suitable components using those libraries.
- **`@SpringBootApplication`:** Enables auto-configuration and component scanning, among other things.
- **Main method:** Calls `SpringApplication.run(...)` to launch the application.

**Interview answer:**

Spring Boot starter dependencies are convenient dependency bundles for specific purposes. For example, `spring-boot-starter-web` brings in the libraries commonly needed for REST APIs, including Spring MVC and an embedded server. Spring Boot's auto-configuration then configures suitable components, reducing manual setup.

## Q6. What happens when `SpringApplication.run()` executes?

**Startup flow:**

```text
main()
   ↓
SpringApplication.run()
   ↓
Prepare configuration
(Component Scanning + Auto-configuration)
   ↓
Create beans and inject dependencies
   ↓
Refresh ApplicationContext
   ↓
Initialize required eager singleton beans
   ↓
Start embedded server for a web application
   ↓
Application ready to handle requests
```

**Remember:** `SpringApplication.run()` starts the application; the `ApplicationContext` manages the beans.

## Q7. `application.properties` vs `application.yml`

Both define application configuration.

**application.properties**
```properties
server.port=8081
spring.datasource.url=jdbc:mysql://localhost:3306/orders
```

**application.yml**
```yaml
server:
  port: 8081
  # The datasource URL can also be configured here.
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/orders
```

- Properties uses key-value entries.
- YAML uses indentation and hierarchical structure.
- Both support externalized configuration.
- Choose based on readability and team convention.

## Q8. Spring Profiles and Environment Variables

### A. Spring Profiles

Profiles allow different configuration or beans for different environments.

**application-dev.properties**
```properties
server.port=8081
payment.mode=test
```

**application-prod.properties**
```properties
server.port=8082
payment.mode=live
```

Activate a profile:

```bash
java -jar order-service.jar --spring.profiles.active=dev
```

For production, use `prod` instead.

**Configuration flow:**

```text
Deployment selects active profile
              ↓
Spring Boot loads shared configuration
and applicable profile-specific configuration
              ↓
Application uses the selected settings
```

### B. `@Profile` for selecting implementations

```java
public interface PaymentGateway {
    void pay();
}
```

```java
@Component
@Profile("dev")
class FakePaymentGateway implements PaymentGateway {
    public void pay() {
        System.out.println("Test payment");
    }
}
```

```java
@Component
@Profile("prod")
class RealPaymentGateway implements PaymentGateway {
    public void pay() {
        System.out.println("Real payment");
    }
}
```

```java
@Service
class PaymentService {
    private final PaymentGateway gateway;

    PaymentService(PaymentGateway gateway) {
        this.gateway = gateway;
    }
}
```

With the appropriate profile active, Spring registers the eligible implementation and injects it into `PaymentService`, assuming no other eligible bean creates ambiguity.

### C. Environment variables and secrets

In `application.properties`:

```properties
spring.datasource.username=${DB_USERNAME}
spring.datasource.password=${DB_PASSWORD}
```

These are placeholders, not actual credentials. Spring resolves them from the available configuration sources.

**Credential flow:**

```text
Deployment environment or secret manager
                  ↓
Provides DB_USERNAME and DB_PASSWORD
                  ↓
Spring Boot resolves ${DB_PASSWORD}
                  ↓
Application uses the resolved credential
```

Values may be supplied through deployment environment variables, Docker configuration, Kubernetes Secrets, or secret managers such as HashiCorp Vault and AWS Secrets Manager.

**Important distinctions:**
- Environment variables are a delivery mechanism for process values.
- Secret managers provide facilities for securely storing and managing secrets.
- Environment variables are not automatically encrypted or secret.
- Never commit real production credentials to Git or log secrets.

### D. Reading configuration in Java

**`@Value` — individual property**

```java
@Value("${payment.api-key}")
private String apiKey;
```

**`@ConfigurationProperties` — related properties**

```java
@ConfigurationProperties(prefix = "payment")
public class PaymentProperties {
    private String apiKey;
    private String mode;

    // Getters and setters
}
```

Register the configuration properties class using `@ConfigurationPropertiesScan` or `@EnableConfigurationProperties`.

Prefer `@ConfigurationProperties` for groups of related settings because it provides structured, type-safe configuration binding.

**Interview distinction:**
- `@Profile` → selects environment-specific beans/configuration.
- Environment variables → supply values to the process.
- `@Value` → injects an individual property.
- `@ConfigurationProperties` → binds a group of related properties.

## Q9. `@Value` vs `@ConfigurationProperties`

| `@Value` | `@ConfigurationProperties` |
|---|---|
| Best for individual properties | Best for related property groups |
| Uses property placeholders | Binds properties under a prefix |
| Simple, direct injection | Structured and type-safe binding |

**Interview answer:** I use `@Value` for a small number of individual values and `@ConfigurationProperties` when managing a group of related application settings.

## Q10. What is `@ConditionalOnProperty`?

**Definition:** `@ConditionalOnProperty` conditionally registers a Spring bean based on a configuration property's value.

Example in `application.properties`:

```properties
notification.email.enabled=true
```

```java
@Component
@ConditionalOnProperty(
    name = "notification.email.enabled",
    havingValue = "true"
)
public class EmailNotificationService {

    public void sendEmail() {
        System.out.println("Email sent");
    }
}
```

**Behavior:**

- Property is `true` → condition matches; the bean can be registered.
- Property is `false` → condition doesn't match.
- Property is missing → condition doesn't match by default because `matchIfMissing` defaults to `false`.

**Interview distinction:**

- `@Profile("prod")` → checks the active environment profile.
- `@ConditionalOnProperty` → checks a configuration property's value.

**Memory trick:** `@Profile` = environment; `@ConditionalOnProperty` = configuration value.

## Q11. Embedded server, executable JAR and WAR

**Executable JAR:** Packages application code and dependencies, including an embedded server for a typical Spring Boot web application.

```bash
java -jar order-service.jar
```

No separate Tomcat installation is normally needed when the JAR includes embedded Tomcat.

**Traditional WAR:** A Web Application Archive commonly deployed to an externally installed servlet container such as Tomcat.

| Executable JAR | Traditional WAR |
|---|---|
| Run using `java -jar` | Traditionally deployed to an external server |
| Embedded server included when configured | External container supplies the server |
| Common for Spring Boot microservices | Common in traditional enterprise deployments |

**Interview answer:**

An executable Spring Boot JAR packages the application with its dependencies and an embedded server, allowing us to run it directly using `java -jar`. A traditional WAR is deployed to an external servlet container such as Tomcat.

**Nuance:** Spring Boot also supports WAR packaging, and a WAR can be executable in suitable configurations.

## Q12. Common startup errors and bean-creation failures

Example:

```text
APPLICATION FAILED TO START
Error creating bean with name 'orderService'
UnsatisfiedDependencyException
```

**Common causes:**
- Required bean is missing.
- Component scanning doesn't discover the class.
- Multiple beans cause ambiguity.
- Configuration properties are missing or invalid.
- A dependency fails during its own initialization.

### Debugging approach — preserve for interviews

1. Read the complete startup error and stack trace.
2. Find the deepest relevant `Caused by` message.
3. Check whether the required bean exists and is eligible for injection.
4. Fix the root cause and restart the application.

### Interview-ready answer — preserve

“When a Spring Boot application fails with `UnsatisfiedDependencyException`, I check the complete stack trace and identify the root cause. Common reasons include a missing bean, incorrect component scanning, multiple beans causing ambiguity, or invalid configuration. I verify bean registration and dependency injection, fix the root cause, and restart the application. A circuit breaker is relevant to remote-service call failures, not ordinary Spring bean-creation failures.”

**Practical example:** If `OrderService` requires `PaymentService`, verify that `PaymentService` is registered as a bean and is eligible for injection. Check whether `@Service`, `@Component`, or an appropriate `@Bean` declaration is present.

---

## Final 60-second revision

- **Spring Boot:** Reduces configuration and simplifies deployment.
- **Auto-configuration:** Checks classpath and conditions, then configures suitable components.
- **Starter:** Supplies libraries.
- **`@SpringBootApplication`:** Enables auto-configuration and component scanning.
- **`SpringApplication.run()`:** Launches the application; `ApplicationContext` manages beans.
- **Profiles:** Environment-specific configuration and bean selection.
- **`@Value`:** Individual property; **`@ConfigurationProperties`:** Related property group.
- **`@ConditionalOnProperty`:** Bean registration depends on a configuration value.
- **Executable JAR:** Runs with an embedded server when packaged/configured for it.
- **`UnsatisfiedDependencyException`:** Read the stack trace, identify the root cause, verify beans/configuration, and fix the issue.
- **Circuit breaker:** Protects remote calls; it does not fix ordinary bean-creation failures.

**Topic 2 complete: 12/12.** Next in your learning plan: continue with the next planned Spring/Hibernate/JPA/Maven/Microservices topic without repeating covered material.
