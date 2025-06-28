## 🧠 **Core Spring Concepts (IOC Container + Dependency Injection)**

### 🔸 Spring Container (IOC)

* **IOC (Inversion of Control)** – Spring manages object creation and wiring.
* **Dependency Injection (DI)** – Objects (beans) are injected instead of manually created.
* **Bean** – A Spring-managed object.
* **Bean Life Cycle** – Initialization → Use → Destruction.
* **ApplicationContext** – Central interface for accessing the Spring container.
* **BeanFactory** – Simpler container than ApplicationContext.

---

### 🔸 Bean Definition and Configuration

* `@Component`, `@Service`, `@Repository`, `@Controller` – Marks classes as beans.
* `@Configuration` – Declares a class that defines beans.
* `@Bean` – Declares a bean inside a config class.
* `@ComponentScan` – Tells Spring where to look for components.
* `@Scope` – Defines bean scope (`singleton`, `prototype`, etc.).
* `@Lazy` – Delays bean initialization until needed.
* `@Primary` – Chooses preferred bean when multiple exist.
* `@Qualifier` – Specifies which bean to inject when multiple match.
* `@Autowired` – Automatically injects a bean.
* `@Value` – Injects values from `properties` files.

---

### 🔸 Environment & Configuration

* **Property Sources** – Read from `application.properties`, `.yml`, system env, etc.
* `@PropertySource` – Load custom property files.
* `Environment` – Interface to access property values.

---

### 🔸 AOP (Aspect-Oriented Programming) – *Still core*

* **Aspect** – Cross-cutting concern (e.g., logging, security).
* `@Aspect` – Marks a class as an aspect.
* `@Before`, `@After`, `@Around` – Advice types.
* `@EnableAspectJAutoProxy` – Enables AOP proxying.

---

### 🔸 Core Utilities

* `ApplicationContextAware`, `BeanNameAware` – Interfaces to access context/bean info.
* `@PostConstruct`, `@PreDestroy` – Bean lifecycle hooks.
* `FactoryBean` – Customize how beans are created.
* `@Profile` – Load beans conditionally based on environment.

---

## 🚀 **Spring Boot Core Concepts**

### 🔹 Core Features

* **Auto-Configuration** – Automatically configures beans based on classpath.
* `@SpringBootApplication` – Main annotation (includes `@Configuration`, `@ComponentScan`, `@EnableAutoConfiguration`).
* **Spring Boot Starters** – Pre-defined dependency sets like:

  * `spring-boot-starter`
  * `spring-boot-starter-web`
  * `spring-boot-starter-test`
* **CommandLineRunner / ApplicationRunner** – Runs code at app startup.
* `application.properties` / `application.yml` – Central config file.
* **Embedded Server** – Tomcat/Jetty/Undertow runs inside your app.
* **Spring Boot DevTools** – Live reload, cache disable, etc.

---

### 🔹 Profiles and Config

* `spring.profiles.active` – Switch between `dev`, `test`, `prod`, etc.
* `@Profile("dev")` – Load beans conditionally per environment.
* `@ConfigurationProperties` – Bind full configuration classes to properties.
