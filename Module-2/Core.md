## **Introduction: The Spring IoC Container and Beans**

Before we jump into annotations, let's understand the fundamental concept:

  * **Inversion of Control (IoC):** Traditionally, objects create their dependencies. In IoC, the framework (Spring) creates and manages the dependencies for you. It *inverts* the control of object creation and lifecycle.
  * **IoC Container:** This is the core of the Spring Framework. It's responsible for instantiating, configuring, and managing the lifecycle of your application's objects, known as **beans**.
  * **Beans:** In Spring, a bean is simply an object that is instantiated, assembled, and managed by the Spring IoC container. These are the building blocks of your application.

Think of the Spring IoC container as a highly organized factory. You tell it what objects you need (your beans), and it takes care of creating them, connecting them, and giving them to you when you ask.

**Why is this important?**

  * **Loose Coupling:** Your objects don't need to know how their dependencies are created.
  * **Testability:** Easier to mock dependencies for testing.
  * **Maintainability:** Changes in dependencies don't ripple through your entire codebase.

-----

## **1. The Core Stereotype Annotations: `@Component`, `@Service`, `@Repository`, `@Controller`**

These annotations are the simplest way to tell Spring, "Hey, this class is a bean\! Please manage it." They are known as **stereotype annotations** because they indicate the typical role or "stereotype" of the class they annotate within your application's architecture.

**Visualizing the Layers:**

Imagine a typical multi-layered application:

```
+-------------------+      +-------------------+      +-------------------+      +-------------------+
|    Presentation   | ---->|    Service      | ---->|    Data Access    | ---->|    Database       |
|    Layer          |      |    Layer          |      |    Layer          |      |                   |
| (e.g., REST APIs) |      | (Business Logic)  |      | (Data Operations) |      |                   |
+-------------------+      +-------------------+      +-------------------+      +-------------------+
  @Controller                @Service                 @Repository
```

### **1.1. `@Component`**

  * **What it means:** A generic stereotype for any Spring-managed component. It's the most general-purpose annotation among the four.
  * **Where to use it:** Use it for any class that doesn't fit into the more specific stereotypes but needs to be managed by Spring. Think of utility classes, helpers, or custom components that don't belong to the service, data, or presentation layers.
  * **How it works:** When Spring performs component scanning, it discovers classes annotated with `@Component` (and its specializations) and registers them as beans in the application context.

**Real Project Code Example:**

Let's say you have a utility class for handling custom logging.

```java
package com.example.app.util;

import org.springframework.stereotype.Component;

@Component
public class CustomLogger {

    public void log(String message) {
        System.out.println("[CUSTOM_LOG] " + message);
    }
}
```

**Typical Use Case:** Generic utility classes, helper components.

-----

### **1.2. `@Service`**

  * **What it means:** Indicates that an annotated class is a "Service" that holds business logic. It's a specialization of `@Component`.
  * **Where to use it:** Primarily used in the **Service Layer**. This layer encapsulates the business rules and orchestrates calls to the data access layer.
  * **Differences from `@Component`:** Semantically, it clarifies the role of the class. Functionally, it's very similar to `@Component` in terms of bean detection. However, future Spring versions or specific Spring Boot starters might add additional behavior to `@Service` (e.g., for transaction management defaults, although explicitly declaring `@Transactional` is usually preferred).

**Real Project Code Example:**

An `OrderService` that handles the business logic for creating and managing orders.

```java
package com.example.app.service;

import com.example.app.repository.OrderRepository;
import com.example.app.model.Order;
import org.springframework.stereotype.Service;

@Service
public class OrderService {

    private final OrderRepository orderRepository; // Dependency injection

    public OrderService(OrderRepository orderRepository) {
        this.orderRepository = orderRepository;
    }

    public Order createOrder(Order order) {
        // Business logic: e.g., validate order, calculate total, apply discounts
        order.setStatus("PENDING");
        return orderRepository.save(order);
    }

    public Order getOrderById(Long id) {
        return orderRepository.findById(id).orElse(null);
    }
}
```

**Typical Use Case:** Business logic, transaction management, orchestrating data access.

-----

### **1.3. `@Repository`**

  * **What it means:** Indicates that an annotated class is a "Repository" that interacts with the database. It's a specialization of `@Component`.
  * **Where to use it:** Exclusively used in the **Data Access Layer (DAO Layer)**. These classes are responsible for performing CRUD (Create, Read, Update, Delete) operations on the database.
  * **Differences from `@Component`:**
      * **Semantics:** Clearly defines the class's role in data persistence.
      * **Exception Translation:** Perhaps the most significant difference. Spring provides `PersistenceExceptionTranslationPostProcessor` which can translate database-specific exceptions (like `SQLException`) into Spring's unchecked `DataAccessException` hierarchy. This makes your data access code more portable and easier to handle. `@Repository` automatically qualifies a bean for this post-processing.

**Real Project Code Example:**

An `OrderRepository` that interacts with the database to save and retrieve orders.

```java
package com.example.app.repository;

import com.example.app.model.Order;
import org.springframework.stereotype.Repository;

import java.util.HashMap;
import java.util.Map;
import java.util.Optional;
import java.util.concurrent.atomic.AtomicLong;

// A simplified in-memory repository for demonstration
@Repository
public class OrderRepository {

    private final Map<Long, Order> orders = new HashMap<>();
    private final AtomicLong counter = new AtomicLong();

    public Order save(Order order) {
        if (order.getId() == null) {
            order.setId(counter.incrementAndGet());
        }
        orders.put(order.getId(), order);
        System.out.println("Order saved: " + order.getId());
        return order;
    }

    public Optional<Order> findById(Long id) {
        return Optional.ofNullable(orders.get(id));
    }

    // ... other CRUD operations
}
```

**Typical Use Case:** Database interaction, DAO classes, ORM integration (JPA, Hibernate).

-----

### **1.4. `@Controller`**

  * **What it means:** Indicates that an annotated class is a "Controller" in the MVC (Model-View-Controller) pattern. It's a specialization of `@Component`.
  * **Where to use it:** Primarily used in the **Presentation Layer**. These classes handle incoming web requests, process user input, and return responses (e.g., HTML views or JSON data for REST APIs).
  * **Differences from `@Component`:**
      * **Web Context:** `@Controller` is specifically designed for web applications. It's picked up by Spring MVC and is capable of handling HTTP requests.
      * **Method Mapping:** Methods within `@Controller` classes can be further annotated with `@RequestMapping`, `@GetMapping`, `@PostMapping`, etc., to map them to specific URLs.
      * **View Resolution:** Typically works with view resolvers to return view names (for traditional web apps). For REST APIs, `@RestController` is a more common and convenient variant.

**Real Project Code Example:**

An `OrderController` that exposes REST endpoints for managing orders.

```java
package com.example.app.controller;

import com.example.app.model.Order;
import com.example.app.service.OrderService;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.stereotype.Controller; // Or @RestController for REST APIs
import org.springframework.web.bind.annotation.*;

@Controller // For a traditional MVC controller returning views
// For a REST API, you'd typically use @RestController, which is a convenience annotation that combines @Controller and @ResponseBody
// @RestController // If you want to return JSON directly
@RequestMapping("/api/orders")
public class OrderController {

    private final OrderService orderService; // Dependency injection

    public OrderController(OrderService orderService) {
        this.orderService = orderService;
    }

    @PostMapping
    public ResponseEntity<Order> createOrder(@RequestBody Order order) {
        Order createdOrder = orderService.createOrder(order);
        return new ResponseEntity<>(createdOrder, HttpStatus.CREATED);
    }

    @GetMapping("/{id}")
    public ResponseEntity<Order> getOrderById(@PathVariable Long id) {
        Order order = orderService.getOrderById(id);
        if (order != null) {
            return ResponseEntity.ok(order);
        } else {
            return ResponseEntity.notFound().build();
        }
    }
}
```

**Typical Use Case:** Handling web requests, building REST APIs, serving HTML pages.

-----

### **Differences Between Stereotype Annotations (Summary):**

| Annotation     | Primary Role                                  | Layer                 | Key Feature(s)                                                                           |
| :------------- | :-------------------------------------------- | :-------------------- | :--------------------------------------------------------------------------------------- |
| `@Component`   | Generic Spring-managed component              | Any (general purpose) | Basic bean detection.                                                                    |
| `@Service`     | Business logic component                      | Service Layer         | Semantic clarity, potential for additional framework behavior (e.g., transaction default). |
| `@Repository`  | Data access component                         | Data Access Layer     | Semantic clarity, automatic exception translation (from DB specific to `DataAccessException`). |
| `@Controller`  | Web request handler (MVC)                     | Presentation Layer    | Semantic clarity, enables Spring MVC features (`@RequestMapping`, etc.).                 |
| `@RestController` | Web request handler returning JSON/XML directly | Presentation Layer    | Convenience annotation combining `@Controller` and `@ResponseBody`.                      |

**Important Note:** All of `@Service`, `@Repository`, and `@Controller` are internally annotated with `@Component`. This means that if Spring's component scanning is enabled, a class annotated with any of these will also be detected as a Spring bean. The specialized annotations provide **semantic meaning** and may offer **additional framework-specific processing**.

**Interview Questions & Answers (Stereotype Annotations):**

1.  **Q:** What is the primary purpose of `@Component`?
      * **A:** `@Component` is a generic stereotype annotation that indicates a class is a Spring-managed component. It tells Spring to detect and register this class as a bean in its application context during component scanning.
2.  **Q:** What's the main difference between `@Service` and `@Component`?
      * **A:** Functionally, both are very similar in terms of bean detection. The main difference is semantic. `@Service` provides a clearer indication that the class holds business logic within the service layer. While `@Component` is general, `@Service` adds specific role clarity.
3.  **Q:** Why should you use `@Repository` instead of `@Component` for your DAO classes?
      * **A:** `@Repository` offers two key advantages: semantic clarity (it clearly states the class's role in data persistence) and, more importantly, it enables Spring's `PersistenceExceptionTranslationPostProcessor` to translate database-specific exceptions into Spring's unified `DataAccessException` hierarchy. This makes error handling more consistent and portable.
4.  **Q:** When would you use `@RestController` over `@Controller`?
      * **A:** Use `@RestController` when building RESTful web services that primarily return data (like JSON or XML) directly in the response body. It's a convenience annotation that combines `@Controller` and `@ResponseBody`, eliminating the need to add `@ResponseBody` to every method. Use `@Controller` for traditional MVC applications that typically return view names.

-----

## **2. `@Configuration + @Bean`: Java-based Configuration**

While stereotype annotations are great for automatically discovering classes, sometimes you need more control. This is where Java-based configuration with `@Configuration` and `@Bean` comes in.

### **2.1. What is a Java-based Configuration Class?**

A Java-based configuration class is a regular Java class annotated with `@Configuration`. It's a Spring-specific mechanism that allows you to define your beans using plain Java methods, rather than XML configuration or relying solely on component scanning.

  * **`@Configuration`:** This annotation tells Spring that a class contains `@Bean` methods. Spring will process this class to generate bean definitions and service requests for those beans at runtime. `@Configuration` classes are themselves Spring beans.
  * **`@Bean`:** This annotation is used on methods within a `@Configuration` class. It tells Spring that the method's return value should be registered as a Spring bean in the application context. The method name typically becomes the bean ID.

### **2.2. How and When to Use `@Bean` to Manually Define Objects**

You use `@Bean` when:

  * You need to configure a third-party library's class as a bean, and you don't have access to its source code to add `@Component`.
  * You need to perform some complex initialization logic before the bean is ready.
  * You want to conditionally create a bean based on certain criteria (e.g., profile-specific beans).
  * You have multiple instances of the same type of bean and need to distinguish them.

**Process:**

1.  Create a class and annotate it with `@Configuration`.
2.  Inside this class, define methods. Each method should:
      * Be annotated with `@Bean`.
      * Return an instance of the class you want to register as a bean.
      * The method name will be the bean ID (unless explicitly specified using `@Bean("myBeanName")`).

**Example: `AppConfig` class creating beans manually**

Let's create a `MessageService` interface and two implementations, `EmailService` and `SMSService`. We'll configure them using `@Configuration` and `@Bean`.

```java
// 1. Define an interface
package com.example.app.service;

public interface MessageService {
    void sendMessage(String message, String recipient);
}
```

```java
// 2. Implementations
package com.example.app.service.impl;

import com.example.app.service.MessageService;
import org.springframework.stereotype.Service; // (Optional: will compare later)

//@Service // Not using this for @Configuration + @Bean example initially
public class EmailService implements MessageService {
    @Override
    public void sendMessage(String message, String recipient) {
        System.out.println("Email sent to " + recipient + ": " + message);
    }
}
```

```java
package com.example.app.service.impl;

import com.example.app.service.MessageService;
import org.springframework.stereotype.Service; // (Optional: will compare later)

//@Service // Not using this for @Configuration + @Bean example initially
public class SMSService implements MessageService {
    @Override
    public void sendMessage(String message, String recipient) {
        System.out.println("SMS sent to " + recipient + ": " + message);
    }
}
```

Now, the configuration class:

```java
// 3. The Configuration Class
package com.example.app.config;

import com.example.app.service.MessageService;
import com.example.app.service.impl.EmailService;
import com.example.app.service.impl.SMSService;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration // Marks this class as a source of bean definitions
public class AppConfig {

    @Bean // The return value of this method will be registered as a Spring bean
    public MessageService emailService() {
        return new EmailService(); // Manual instantiation
    }

    @Bean
    public MessageService smsService() {
        return new SMSService(); // Manual instantiation
    }

    // You can also define beans that depend on other beans defined in the same config class
    @Bean
    public MyMessageSender myMessageSender() {
        // Here we are injecting the emailService bean into MyMessageSender
        return new MyMessageSender(emailService()); // Calling the @Bean method directly
    }
}
```

```java
// A class that uses MessageService
package com.example.app.config;

import com.example.app.service.MessageService;

public class MyMessageSender {
    private final MessageService messageService;

    public MyMessageSender(MessageService messageService) {
        this.messageService = messageService;
    }

    public void sendNotification(String message, String recipient) {
        messageService.sendMessage(message, recipient);
    }
}
```

**Running the example:**

```java
package com.example.app;

import com.example.app.config.AppConfig;
import com.example.app.config.MyMessageSender;
import com.example.app.service.MessageService;
import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;

public class MainApplication {
    public static void main(String[] args) {
        // Create a Spring application context based on our AppConfig
        ApplicationContext context = new AnnotationConfigApplicationContext(AppConfig.class);

        // Retrieve beans by their method names (default bean IDs)
        MessageService emailService = context.getBean("emailService", MessageService.class);
        emailService.sendMessage("Hello via Email!", "john.doe@example.com");

        MessageService smsService = context.getBean("smsService", MessageService.class);
        smsService.sendMessage("Hello via SMS!", "123-456-7890");

        MyMessageSender sender = context.getBean("myMessageSender", MyMessageSender.class);
        sender.sendNotification("Urgent update!", "jane.doe@example.com");

        // Close the context
        ((AnnotationConfigApplicationContext) context).close();
    }
}
```

**Output:**

```
Email sent to john.doe@example.com: Hello via Email!
SMS sent to 123-456-7890: Hello via SMS!
Email sent to jane.doe@example.com: Urgent update!
```

### **2.3. Compare with Annotation-based `@Component` Scanning**

| Feature/Aspect      | `@Component` Scanning                                         | `@Configuration + @Bean` (Java-based Config)                              |
| :------------------ | :------------------------------------------------------------ | :------------------------------------------------------------------------ |
| **Discovery** | Automatic: Spring scans predefined packages for annotated classes. | Explicit: You explicitly define beans within `@Configuration` methods.    |
| **Control** | Less control: Spring decides how to instantiate and wire.     | More control: You write the instantiation logic directly.                 |
| **Use Cases** | Your own classes, standard Spring components.                 | Third-party classes, complex initialization, conditional bean creation.   |
| **Readability** | Often cleaner for simple cases, less boilerplate.             | More verbose for simple cases, but very clear for complex logic.          |
| **Coupling** | Annotation-based.                                             | Java code based, can be more decoupled from Spring annotations in the bean itself. |
| **When to use** | Default for most of your application components.              | When fine-grained control over bean creation or external libraries are involved. |

**Memory Map / How Spring sees it:**

**Before (Manual Object Creation):**

```
Your Code:
  new EmailService();
  new SMSService();
  new MyMessageSender(new EmailService());

Memory:
  [EmailService Object 1]
  [SMSService Object 1]
  [EmailService Object 2]  <-- A new EmailService is created for MyMessageSender
  [MyMessageSender Object]
```

**After (Spring-managed with `@Configuration + @Bean`):**

```
Spring IoC Container:
  +--------------------------------+
  | ApplicationContext             |
  +--------------------------------+
  | emailService (singleton)  ----|---+  (Reference)
  | smsService (singleton)    ----|-----+
  | myMessageSender (singleton) --|-------+
  +--------------------------------+       |
                                           |
Memory:                                    |
  [EmailService Object 1] <----------------+
  [SMSService Object 1]   <----------------+
  [MyMessageSender Object] <---------------+
    - has a reference to [EmailService Object 1]
```

Notice how `emailService()` method inside `AppConfig` is called multiple times but it returns the same instance to `myMessageSender` because beans are singletons by default.

**Interview Questions & Answers (`@Configuration` + `@Bean`):**

1.  **Q:** What is a `@Configuration` class in Spring?
      * **A:** A `@Configuration` class is a Java class annotated with `@Configuration` that Spring uses as a source of bean definitions. It's an alternative to XML-based configuration, allowing you to define beans using plain Java methods annotated with `@Bean`.
2.  **Q:** When would you prefer `@Bean` over stereotype annotations like `@Component`?
      * **A:** You'd prefer `@Bean` when you need explicit control over bean creation, such as configuring third-party classes, performing complex initialization logic, or when creating multiple instances of the same type of bean with different configurations. Stereotype annotations are for auto-detection of your own classes.
3.  **Q:** Can a `@Configuration` class also be `@Component` scanned?
      * **A:** Yes, a `@Configuration` class itself is a `@Component` (it's meta-annotated with `@Component`). This means if your component scan includes the package of your configuration class, Spring will discover and process it.

-----

## **3. `@ComponentScan`**

`@ComponentScan` is the magic behind how Spring Boot automatically finds your beans annotated with `@Component`, `@Service`, `@Repository`, and `@Controller`.

### **3.1. What it does, how to use it, and why it's needed**

  * **What it does:** `@ComponentScan` instructs Spring to scan specified packages for classes annotated with Spring's stereotype annotations (`@Component`, `@Service`, `@Repository`, `@Controller`, `@Configuration`, etc.) and register them as beans in the Spring IoC container.
  * **How to use it:** You typically use it on your main application class (the one with `@SpringBootApplication`) or on a `@Configuration` class.
  * **Why it's needed:** Without `@ComponentScan`, Spring wouldn't know where to look for your components, and they wouldn't be registered as beans, meaning you couldn't `@Autowired` them or use their Spring-managed features. It automates the process of bean definition.

### **3.2. Explain default behavior vs custom base packages**

  * **Default Behavior (Spring Boot):** When you use `@SpringBootApplication` (which includes `@ComponentScan` by default), Spring Boot automatically scans the package of your main application class and all its sub-packages. This is why you often don't explicitly add `@ComponentScan` in Spring Boot applications.

      * **Example (Main application class):**
        ```java
        // com.example.myproject.MyApplication.java
        package com.example.myproject;

        import org.springframework.boot.SpringApplication;
        import org.springframework.boot.autoconfigure.SpringBootApplication;

        @SpringBootApplication // This includes @ComponentScan(basePackages = "com.example.myproject") implicitly
        public class MyApplication {
            public static void main(String[] args) {
                SpringApplication.run(MyApplication.class, args);
            }
        }
        ```
        In this case, Spring will scan `com.example.myproject` and all its sub-packages (e.g., `com.example.myproject.service`, `com.example.myproject.repository`).

  * **Custom Base Packages:** If your beans are in packages outside of the default scan path, or if you want to be more explicit about what to scan, you can specify `basePackages` or `basePackageClasses` attributes.

      * **`basePackages` (String array):** Specifies an array of package names to scan.

        ```java
        @Configuration
        @ComponentScan(basePackages = {"com.example.app.service", "com.example.app.repository"})
        public class AppConfig {
            // ... bean definitions if any
        }
        ```

      * **`basePackageClasses` (Class array):** Specifies an array of classes. Spring will scan the packages where these classes reside. This is safer as it's refactoring-friendly (if you rename a package, the class name will still be valid).

        ```java
        @Configuration
        @ComponentScan(basePackageClasses = {EmailService.class, OrderRepository.class}) // Scan packages of these classes
        public class AppConfig {
            // ...
        }
        ```

        Or combined with `@SpringBootApplication`:

        ```java
        // com.example.myproject.MyApplication.java
        package com.example.myproject;

        import com.another.library.external.ExternalComponent; // Assume this is a bean you need to scan
        import org.springframework.boot.SpringApplication;
        import org.springframework.boot.autoconfigure.SpringBootApplication;
        import org.springframework.context.annotation.ComponentScan;

        @SpringBootApplication
        @ComponentScan(basePackageClasses = {MyApplication.class, ExternalComponent.class}) // Scan current package + external component's package
        public class MyApplication {
            public static void main(String[] args) {
                SpringApplication.run(MyApplication.class, args);
            }
        }
        ```

### **3.3. Code examples that demonstrate scanning depth and filtering**

**Scanning Depth:**

Spring's component scanning is recursive. If you specify `com.example`, it will scan `com.example.service`, `com.example.repository`, `com.example.controller`, and any further sub-packages within them.

**Filtering (Advanced):**

You can fine-tune what to include or exclude during scanning using `includeFilters` and `excludeFilters`. This is useful for very specific scenarios, though less common for typical applications.

  * **`includeFilters`:** Only include types that match the filter criteria.
  * **`excludeFilters`:** Exclude types that match the filter criteria.

**Example (Excluding specific components):**

Let's say you have `OldService` in `com.example.app.service` that you don't want Spring to pick up.

```java
// com.example.app.service.OldService.java
package com.example.app.service;

import org.springframework.stereotype.Service;

@Service
public class OldService {
    public void doOldStuff() {
        System.out.println("Doing old stuff.");
    }
}
```

Now, your configuration:

```java
package com.example.app.config;

import com.example.app.service.OldService;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.FilterType;

@Configuration
@ComponentScan(basePackages = "com.example.app",
        excludeFilters = @ComponentScan.Filter(type = FilterType.ASSIGNABLE_TYPE, classes = OldService.class))
public class MyFilteredConfig {
    // Spring will scan 'com.example.app' but will exclude OldService
}
```

**Example (Including only specific components - less common):**

```java
package com.example.app.config;

import com.example.app.service.EmailService;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.FilterType;

@Configuration
@ComponentScan(basePackages = "com.example.app",
        useDefaultFilters = false, // Must set to false to use only include filters
        includeFilters = @ComponentScan.Filter(type = FilterType.ASSIGNABLE_TYPE, classes = EmailService.class))
public class MySpecificConfig {
    // Only EmailService will be scanned from com.example.app
}
```

**Diagram: `@ComponentScan` Flow**

```
+-----------------------------------+
|      Spring Application Start     |
+-----------------------------------+
        |
        V
+-----------------------------------+
|      Locate @SpringBootApplication|
|      (or @Configuration)          |
+-----------------------------------+
        |
        V
+-----------------------------------+
|      Read @ComponentScan          |
|      (or implicit default)        |
+-----------------------------------+
        |
        V
+-----------------------------------+
|      Determine Base Packages      |
|      (e.g., com.example.app)      |
+-----------------------------------+
        |
        V
+-----------------------------------+
|      Begin Scanning Packages      |
|      (Recursively through dirs)   |
+-----------------------------------+
        |     Found a class?
        |      /    \
        |     V      X
        |   Is it annotated with
        |   @Component/@Service/
        |   @Repository/@Controller/
        |   @Configuration?
        |      /    \
        |     V      X
        |   Does it match
        |   include/exclude filters?
        |      /    \
        |     V      X
+---------------------+    +---------------------+
| Register as Spring  |    |     Ignore Class    |
|       Bean          |    |                     |
+---------------------+    +---------------------+
        |
        V
+-----------------------------------+
|      Beans available in IoC       |
|         Container for Injection   |
+-----------------------------------+
```

**Interview Questions & Answers (`@ComponentScan`):**

1.  **Q:** What is the primary function of `@ComponentScan`?
      * **A:** `@ComponentScan` instructs Spring to automatically discover and register beans in the application context. It scans specified packages for classes annotated with stereotype annotations like `@Component`, `@Service`, `@Repository`, and `@Controller`.
2.  **Q:** Why don't we often see `@ComponentScan` explicitly in Spring Boot applications?
      * **A:** Because `@SpringBootApplication` is a meta-annotation that includes `@ComponentScan` by default. It automatically configures component scanning to happen from the package of the main application class and its sub-packages.
3.  **Q:** How can you scan beans located in a different package than your main application class?
      * **A:** You can use the `basePackages` attribute of `@ComponentScan` to specify an array of package names, or `basePackageClasses` to specify an array of classes whose packages should be scanned.

-----

## **4. `@Scope`**

`@Scope` defines the lifecycle and visibility of a Spring bean. By default, Spring beans are `singleton`.

### **4.1. Explain bean scopes: `singleton`, `prototype`, `request`, etc.**

Spring supports several scopes:

  * **`singleton` (Default):**

      * **Behavior:** Only **one single instance** of the bean is created per Spring IoC container. Every time you request this bean, Spring returns the exact same instance.
      * **Use Cases:** Stateless services (e.g., `OrderService`, `UserRepository`), utility classes, controllers. Most beans in a typical application are singletons.
      * **Lifecycle:** Created when the application context starts (or lazily, if specified). Managed completely by the container. Destroyed when the container shuts down.

  * **`prototype`:**

      * **Behavior:** A **new instance** of the bean is created every time it is requested from the Spring IoC container.
      * **Use Cases:** Statefu objects that need to maintain unique data per client/operation, or when you need a fresh instance every time (e.g., a `ShoppingCart` for each user, or a `TaskExecutor` that runs a specific task).
      * **Lifecycle:** Created on demand when requested. Spring hands over the new instance to the client and doesn't manage its full lifecycle (e.g., it won't call `destroy` methods on prototype beans). You are responsible for cleaning up prototype beans.

  * **`request` (Web-aware scope):**

      * **Behavior:** A new instance of the bean is created for **each HTTP request**. It lives only for the duration of that single web request.
      * **Use Cases:** Storing request-specific data that should not be shared across different requests (e.g., user details specific to a current request).
      * **Availability:** Only available in a web-aware Spring `ApplicationContext` (e.g., `WebApplicationContext`).

  * **`session` (Web-aware scope):**

      * **Behavior:** A new instance of the bean is created for **each HTTP session**. It lives for the duration of the user's session.
      * **Use Cases:** Storing user-specific data that needs to persist across multiple requests within the same session (e.g., logged-in user details, shopping cart state across pages).
      * **Availability:** Only available in a web-aware Spring `ApplicationContext`.

  * **`application` (Web-aware scope):**

      * **Behavior:** A single instance of the bean is created for the entire web application lifecycle. It's essentially a `singleton` but specific to the `ServletContext`.
      * **Use Cases:** Global application-level data or resources that are shared by all users and requests.
      * **Availability:** Only available in a web-aware Spring `ApplicationContext`.

  * **`websocket` (Web-aware scope):**

      * **Behavior:** A single instance of the bean is created per WebSocket session.
      * **Use Cases:** Storing data specific to a WebSocket connection.

### **4.2. Code samples showing scope behavior**

Let's define a simple `Counter` bean.

```java
package com.example.app.bean;

import org.springframework.context.annotation.Scope;
import org.springframework.stereotype.Component;

// Default scope is singleton, so @Scope("singleton") is redundant but shown for clarity
@Component
@Scope("singleton")
public class SingletonCounter {
    private int count = 0;

    public void increment() {
        count++;
    }

    public int getCount() {
        return count;
    }
}
```

```java
package com.example.app.bean;

import org.springframework.context.annotation.Scope;
import org.springframework.stereotype.Component;

@Component
@Scope("prototype")
public class PrototypeCounter {
    private int count = 0;

    public void increment() {
        count++;
    }

    public int getCount() {
        return count;
    }
}
```

**Testing the scopes:**

```java
package com.example.app;

import com.example.app.bean.PrototypeCounter;
import com.example.app.bean.SingletonCounter;
import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

@Configuration
@ComponentScan(basePackages = "com.example.app.bean")
public class ScopeDemoApplication {

    public static void main(String[] args) {
        ApplicationContext context = new AnnotationConfigApplicationContext(ScopeDemoApplication.class);

        System.out.println("--- Singleton Scope Demo ---");
        SingletonCounter sc1 = context.getBean(SingletonCounter.class);
        sc1.increment();
        System.out.println("SingletonCounter 1 count: " + sc1.getCount()); // Expected: 1

        SingletonCounter sc2 = context.getBean(SingletonCounter.class);
        sc2.increment();
        System.out.println("SingletonCounter 2 count: " + sc2.getCount()); // Expected: 2 (same instance)
        System.out.println("Are sc1 and sc2 the same instance? " + (sc1 == sc2)); // Expected: true

        System.out.println("\n--- Prototype Scope Demo ---");
        PrototypeCounter pc1 = context.getBean(PrototypeCounter.class);
        pc1.increment();
        System.out.println("PrototypeCounter 1 count: " + pc1.getCount()); // Expected: 1

        PrototypeCounter pc2 = context.getBean(PrototypeCounter.class);
        pc2.increment();
        System.out.println("PrototypeCounter 2 count: " + pc2.getCount()); // Expected: 1 (new instance)
        System.out.println("Are pc1 and pc2 the same instance? " + (pc1 == pc2)); // Expected: false

        ((AnnotationConfigApplicationContext) context).close();
    }
}
```

**Output:**

```
--- Singleton Scope Demo ---
SingletonCounter 1 count: 1
SingletonCounter 2 count: 2
Are sc1 and sc2 the same instance? true

--- Prototype Scope Demo ---
PrototypeCounter 1 count: 1
PrototypeCounter 2 count: 1
Are pc1 and pc2 the same instance? false
```

### **4.3. Diagrams or Lifecycle Flow for `singleton` vs `prototype`**

**Singleton Lifecycle:**

```
[Spring IoC Container Starts]
        |
        V
[Bean Definition Found]
        |
        V
[Instantiate Bean (once)]
        |
        V
[Populate Properties / Set Dependencies]
        |
        V
[Call BeanNameAware.setBeanName()]
[Call BeanFactoryAware.setBeanFactory()]
[Call ApplicationContextAware.setApplicationContext()]
        |
        V
[Call @PostConstruct method (if any)]
        |
        V
[Bean is Ready for Use (stored in container's cache)]
        |
        |  (Multiple requests for the bean)
        |---------------------------------------------> [Return the SAME instance]
        |
[Spring IoC Container Shuts Down]
        |
        V
[Call @PreDestroy method (if any)]
        |
        V
[Destroy Bean Instance]
```

**Prototype Lifecycle:**

```
[Spring IoC Container Starts]
        |
        V
[Bean Definition Found]
        |
        | (No instantiation yet, definition is just registered)
        |
[Request for Bean from Container]
        |
        V
[Instantiate NEW Bean Instance (for each request)]
        |
        V
[Populate Properties / Set Dependencies]
        |
        V
[Call BeanNameAware.setBeanName()]
[Call BeanFactoryAware.setBeanFactory()]
[Call ApplicationContextAware.setApplicationContext()]
        |
        V
[Call @PostConstruct method (if any)]
        |
        V
[Bean is Ready for Use (handed over to client)]
        |
        | (Spring relinquishes control)
        V
[Client is Responsible for Lifecycle/Cleanup]
        |
        V
[Bean is NOT destroyed by Spring container shutdown]
```

**Interview Questions & Answers (`@Scope`):**

1.  **Q:** What is the default scope for Spring beans, and what does it mean?
      * **A:** The default scope is `singleton`. It means that for each Spring IoC container, only one single instance of that bean is created, and all subsequent requests for that bean will return the exact same instance.
2.  **Q:** When would you use a `prototype` scope instead of `singleton`?
      * **A:** Use `prototype` when you need a fresh, independent instance of a bean every time it's requested. This is suitable for stateful beans where each client needs its own unique copy to avoid data corruption (e.g., a shopping cart, a task object).
3.  **Q:** What's the key difference in lifecycle management between `singleton` and `prototype` beans?
      * **A:** Spring manages the complete lifecycle of `singleton` beans (creation, dependency injection, initialization, destruction). For `prototype` beans, Spring only handles creation and dependency injection. After handing over the instance, Spring relinquishes control, and you are responsible for managing its destruction or cleanup.
4.  **Q:** Can you name some web-aware scopes in Spring?
      * **A:** `request`, `session`, and `application` are common web-aware scopes. `websocket` is also available for WebSocket applications.

-----

## **5. `@Lazy`**

By default, Spring singleton beans are eagerly initialized, meaning they are created and wired up during application startup. `@Lazy` allows you to change this behavior.

### **5.1. What lazy initialization is and when to use it**

  * **Lazy Initialization:** Means that a Spring bean is not created until it is actually needed (i.e., when it's first requested or injected into another bean).
  * **When to use it:**
      * **Performance:** If you have many beans, and some are rarely used, lazy loading can speed up application startup time by deferring their creation.
      * **Resource Management:** If a bean consumes significant resources (e.g., database connections, file handles) but isn't always needed immediately, lazy loading can conserve resources.
      * **Circular Dependencies (Potential Help):** In some complex circular dependency scenarios, `@Lazy` can break the cycle by deferring the injection of one of the beans involved.

### **5.2. Show how it affects bean instantiation**

Let's illustrate with an example.

```java
package com.example.app.bean;

import org.springframework.context.annotation.Lazy;
import org.springframework.stereotype.Component;

@Component
public class EagerBean {
    public EagerBean() {
        System.out.println("EagerBean instantiated!");
    }

    public void doSomething() {
        System.out.println("EagerBean doing something.");
    }
}
```

```java
package com.example.app.bean;

import org.springframework.context.annotation.Lazy;
import org.springframework.stereotype.Component;

@Component
@Lazy // This bean will be instantiated only when needed
public class LazyBean {
    public LazyBean() {
        System.out.println("LazyBean instantiated!");
    }

    public void doSomething() {
        System.out.println("LazyBean doing something.");
    }
}
```

```java
package com.example.app.service;

import com.example.app.bean.EagerBean;
import com.example.app.bean.LazyBean;
import org.springframework.stereotype.Service;

@Service
public class MyService {
    private final EagerBean eagerBean;
    private final LazyBean lazyBean;

    // Both beans are injected here
    public MyService(EagerBean eagerBean, LazyBean lazyBean) {
        this.eagerBean = eagerBean;
        this.lazyBean = lazyBean;
        System.out.println("MyService instantiated and dependencies injected.");
    }

    public void triggerLazyBean() {
        System.out.println("Triggering LazyBean...");
        lazyBean.doSomething(); // This is where LazyBean will be instantiated
    }
}
```

**Running the example:**

```java
package com.example.app;

import com.example.app.service.MyService;
import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

@Configuration
@ComponentScan(basePackages = {"com.example.app.bean", "com.example.app.service"})
public class LazyDemoApplication {

    public static void main(String[] args) {
        System.out.println("Starting Spring context...");
        ApplicationContext context = new AnnotationConfigApplicationContext(LazyDemoApplication.class);
        System.out.println("Spring context started.");

        System.out.println("\nRequesting MyService...");
        MyService myService = context.getBean(MyService.class);

        System.out.println("\nCalling triggerLazyBean on MyService...");
        myService.triggerLazyBean();

        ((AnnotationConfigApplicationContext) context).close();
    }
}
```

**Output:**

```
Starting Spring context...
EagerBean instantiated!
MyService instantiated and dependencies injected.
Spring context started.

Requesting MyService...

Calling triggerLazyBean on MyService...
LazyBean instantiated!
LazyBean doing something.
```

**Observation:** `EagerBean` is instantiated when the context starts. `LazyBean` is only instantiated when `triggerLazyBean()` is called on `MyService`, which is when `lazyBean.doSomething()` actually tries to use `LazyBean`.

### **5.3. Explain how it helps with circular dependencies and startup performance**

  * **Circular Dependencies:**

      * A circular dependency occurs when Bean A depends on Bean B, and Bean B depends on Bean A.
      * Spring usually tries to resolve these by creating a partially constructed instance. However, in some cases, especially with constructor injection, it can lead to `BeanCurrentlyInCreationException`.
      * `@Lazy` can help by deferring the injection of one of the beans. If Bean A needs Bean B, and Bean B is `@Lazy`, Spring can instantiate A without immediately resolving B. When A later tries to use B, B will then be instantiated. This breaks the immediate cycle during object construction.
      * **Caveat:** While `@Lazy` *can* help, it's generally better to refactor your design to avoid circular dependencies if possible, as they often indicate a design flaw.

  * **Startup Performance:**

      * For applications with a large number of beans, especially those that are rarely used or are resource-heavy, eager initialization can significantly slow down application startup.
      * By annotating such beans with `@Lazy`, their instantiation is deferred until they are actually needed. This can result in a faster initial startup, making the application seem more responsive.
      * **Trade-off:** The first time a lazy bean is accessed, there will be a slight delay as it's instantiated. This might not be noticeable for most beans, but for very complex ones, it could impact the first user interaction.

**Interview Questions & Answers (`@Lazy`):**

1.  **Q:** What does `@Lazy` do in Spring?
      * **A:** `@Lazy` defers the initialization of a Spring bean. Instead of being created during application startup, a lazy-initialized bean is created only when it is first requested or when it's injected into another bean.
2.  **Q:** What are the main benefits of using `@Lazy`?
      * **A:** The main benefits are improved application startup performance (by not creating all beans upfront) and, in some cases, it can help resolve complex circular dependency issues by deferring injection.
3.  **Q:** What is a potential downside of using `@Lazy` excessively?
      * **A:** The main downside is that the first time a lazy bean is accessed, there will be a small delay for its instantiation. If many critical beans are lazy, this could lead to a less responsive initial user experience, and potential configuration errors might only surface at runtime when the lazy bean is accessed, rather than at startup.

-----

## **6. `@Primary` vs `@Qualifier`**

These annotations become crucial when you have multiple beans of the **same type** in your Spring application context, and you need to specify which one to inject.

### **6.1. When multiple beans of the same type exist**

Imagine you have two different implementations of `MessageService`: `EmailService` and `SMSService`, both implementing the `MessageService` interface.

```java
// Re-using our MessageService and implementations
// com.example.app.service.MessageService
// com.example.app.service.impl.EmailService
// com.example.app.service.impl.SMSService
```

Now, if you try to `@Autowired` `MessageService` directly without further specification:

```java
package com.example.app.service;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

@Service
public class NotificationService {

    @Autowired
    private MessageService messageService; // PROBLEM: Which MessageService? EmailService or SMSService?

    public void sendNotification(String message, String recipient) {
        messageService.sendMessage(message, recipient);
    }
}
```

Spring will throw a `NoUniqueBeanDefinitionException` because it doesn't know which `MessageService` to inject. This is where `@Primary` and `@Qualifier` come to the rescue.

### **6.2. How to use `@Primary` to declare a default**

  * **`@Primary`:** Designates a single bean as the **primary candidate** when multiple beans of the same type are available. If Spring finds multiple beans of a requested type, and one is marked `@Primary`, that one will be chosen by default for auto-wiring.

**Example:** Make `EmailService` the default `MessageService`.

```java
package com.example.app.service.impl;

import com.example.app.service.MessageService;
import org.springframework.context.annotation.Primary;
import org.springframework.stereotype.Service;

@Service
@Primary // This EmailService will be the default when MessageService is autowired
public class EmailService implements MessageService {
    @Override
    public void sendMessage(String message, String recipient) {
        System.out.println("[Primary] Email sent to " + recipient + ": " + message);
    }
}
```

```java
package com.example.app.service.impl;

import com.example.app.service.MessageService;
import org.springframework.stereotype.Service;

@Service
public class SMSService implements MessageService {
    @Override
    public void sendMessage(String message, String recipient) {
        System.out.println("[Secondary] SMS sent to " + recipient + ": " + message);
    }
}
```

Now, `NotificationService` will successfully inject `EmailService`:

```java
package com.example.app.service;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

@Service
public class NotificationService {

    @Autowired
    private MessageService messageService; // Now automatically resolves to EmailService due to @Primary

    public void sendNotification(String message, String recipient) {
        System.out.println("Using: " + messageService.getClass().getSimpleName());
        messageService.sendMessage(message, recipient);
    }
}
```

### **6.3. How `@Qualifier` helps to specify a particular bean**

  * **`@Qualifier`:** Provides a way to specifically name the bean you want to inject when there are multiple beans of the same type. It works in conjunction with `@Autowired`. The qualifier name usually matches the bean's name (which is often the class name converted to camelCase, or the method name for `@Bean` defined beans).

**Example:** Explicitly choose `SMSService` or `EmailService`.

```java
package com.example.app.service;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.stereotype.Service;

@Service
public class AnotherNotificationService {

    // Inject the SMSService specifically
    @Autowired
    @Qualifier("SMSService") // Or "smsService" if using @Bean method name as ID
    private MessageService smsMessageService;

    // Inject the EmailService specifically
    @Autowired
    @Qualifier("emailService") // Or "emailService"
    private MessageService emailMessageService;

    public void sendSmsNotification(String message, String recipient) {
        System.out.println("Using SMS service:");
        smsMessageService.sendMessage(message, recipient);
    }

    public void sendEmailNotification(String message, String recipient) {
        System.out.println("Using Email service:");
        emailMessageService.sendMessage(message, recipient);
    }
}
```

**Note:** When using stereotype annotations like `@Service`, the default bean name is the class name with the first letter lowercased (e.g., `EmailService` becomes `emailService`).

**Using `@Bean` with `@Qualifier`:**

If you define your beans using `@Configuration + @Bean`, the method name is the default bean ID. You can also explicitly set the name:

```java
package com.example.app.config;

import com.example.app.service.MessageService;
import com.example.app.service.impl.EmailService;
import com.example.app.service.impl.SMSService;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Primary;

@Configuration
public class MessageConfig {

    @Bean
    @Primary // Make this the primary MessageService
    public MessageService defaultEmailService() {
        return new EmailService();
    }

    @Bean("smsGateService") // Explicitly naming the bean
    public MessageService smsService() {
        return new SMSService();
    }
}
```

Then, you would qualify with `"defaultEmailService"` or `"smsGateService"`.

```java
package com.example.app.service;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.stereotype.Service;

@Service
public class AdvancedNotificationService {

    @Autowired
    private MessageService defaultService; // This will pick the @Primary 'defaultEmailService'

    @Autowired
    @Qualifier("smsGateService")
    private MessageService smsSpecificService;

    public void sendDefault(String message, String recipient) {
        defaultService.sendMessage(message, recipient);
    }

    public void sendViaSmsGateway(String message, String recipient) {
        smsSpecificService.sendMessage(message, recipient);
    }
}
```

### **6.4. Code snippets and common pitfalls**

**Common Pitfalls:**

1.  **Forgetting `@Primary` or `@Qualifier`:** Leads to `NoUniqueBeanDefinitionException` when multiple beans of the same type exist.
2.  **Incorrect Qualifier Name:** If the `@Qualifier` value doesn't match an existing bean name, Spring will throw `NoSuchBeanDefinitionException`.
3.  **Using `@Primary` with too many beans:** `@Primary` is for setting a *default*. If you have many beans of the same type and need to pick specific ones frequently, `@Qualifier` is more appropriate. Don't overuse `@Primary`.
4.  **Mixing `@Primary` and `@Qualifier` on the same injection point:** This doesn't make sense; `@Qualifier` takes precedence if both are present.

**Diagram: Bean Resolution Flow**

```
[Request for Bean of Type X]
        |
        V
+-------------------------------+
| Are there multiple beans of   |
|      Type X in container?     |
+-------------------------------+
        | Yes / No
        |
        V
      No:                 Yes:
[Inject the single bean]  +--------------------------------+
                          | Is there exactly one bean with |
                          |       @Primary?                |
                          +--------------------------------+
                                  | Yes / No
                                  |
                                  V
                                Yes:                     No:
                                [Inject @Primary bean]   +--------------------------------+
                                                         | Is there an @Qualifier         |
                                                         |       on injection point?      |
                                                         +--------------------------------+
                                                                 | Yes / No
                                                                 |
                                                                 V
                                                               Yes:                     No:
                                                               [Inject bean matching    |
                                                                  Qualifier name]       |
                                                                                        V
                                                                           [NoUniqueBeanDefinitionException]
```

**Interview Questions & Answers (`@Primary` vs `@Qualifier`):**

1.  **Q:** When do you need to use `@Primary` or `@Qualifier`?
      * **A:** You need them when you have multiple beans of the same type registered in the Spring IoC container, and Spring needs guidance on which specific bean to inject during auto-wiring.
2.  **Q:** What's the main difference between `@Primary` and `@Qualifier`?
      * **A:** `@Primary` designates a single default bean among multiple candidates of the same type. Spring will automatically inject this primary bean if no specific qualifier is provided. `@Qualifier`, on the other hand, provides a way to explicitly specify which bean to inject by its unique name, overriding the primary choice if present.
3.  **Q:** If a bean is marked `@Primary`, but an injection point uses `@Qualifier` for a different bean of the same type, which one wins?
      * **A:** `@Qualifier` always takes precedence over `@Primary`. If a qualifier is present on the injection point, Spring will ignore any `@Primary` declarations and inject the bean matching the qualifier name.

-----

## **7. `@Autowired`**

`@Autowired` is the cornerstone of Spring's **Dependency Injection (DI)** mechanism. It allows Spring to automatically "wire" dependencies between your beans.

### **7.1. How automatic dependency injection works**

When Spring creates a bean, it looks for fields, constructors, or setter methods annotated with `@Autowired`. If found, Spring automatically searches its application context for a compatible bean of the required type and injects it.

**Process:**

1.  Spring identifies a bean that needs dependencies.
2.  It finds an `@Autowired` annotation on a constructor, setter, or field.
3.  It determines the type of the dependency needed.
4.  It searches the IoC container for a bean that matches that type.
5.  If a unique match is found, it injects the dependency.
6.  If multiple matches are found, it uses `@Primary` or `@Qualifier` rules to resolve.
7.  If no match or no unique match without guidance, it throws an exception.

### **7.2. Where and how to use field, setter, and constructor injection**

There are three main ways to use `@Autowired`:

1.  **Constructor Injection (Recommended):**

      * **How:** Apply `@Autowired` to the constructor of your class.
      * **Pros:**
          * **Immutability:** Dependencies can be `final`, ensuring they are set once and never changed.
          * **Testability:** Easier to create instances of the class in unit tests without Spring, as dependencies are explicit constructor arguments.
          * **Guaranteed Injection:** Ensures all required dependencies are provided at object creation, making the object valid from the start.
          * **No Circular Dependency Issues (Immediate):** If a circular dependency exists with constructor injection, Spring will fail fast at startup, which is good for identifying design flaws early.
      * **Cons:** Can lead to "constructor bloat" if a class has many dependencies.

    <!-- end list -->

    ```java
    package com.example.app.service;

    import org.springframework.stereotype.Service;

    @Service
    public class ProductService {
        private final ProductRepository productRepository; // Final and immutable

        //@Autowired // Optional in Spring 4.3+ for single constructor
        public ProductService(ProductRepository productRepository) {
            this.productRepository = productRepository;
            System.out.println("ProductService: Constructor Injection");
        }

        // ... methods using productRepository
    }
    ```

2.  **Setter Injection:**

      * **How:** Apply `@Autowired` to a setter method.
      * **Pros:**
          * **Optional Dependencies:** Can be used for optional dependencies, as Spring won't throw an error if the bean isn't found (if `required = false`).
          * **Mutability:** Allows changing dependencies after object creation (though often undesirable for services).
      * **Cons:** Object can be in an inconsistent state before all setters are called. Not ideal for mandatory dependencies.

    <!-- end list -->

    ```java
    package com.example.app.service;

    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.stereotype.Service;

    @Service
    public class InventoryService {
        private InventoryRepository inventoryRepository;

        @Autowired
        public void setInventoryRepository(InventoryRepository inventoryRepository) {
            this.inventoryRepository = inventoryRepository;
            System.out.println("InventoryService: Setter Injection");
        }

        // ... methods using inventoryRepository
    }
    ```

3.  **Field Injection:**

      * **How:** Apply `@Autowired` directly to a field.
      * **Pros:**
          * **Concise:** Less boilerplate code.
      * **Cons:**
          * **Not Testable:** Hard to unit test without Spring's container because you can't easily mock private fields.
          * **Immutability:** Cannot make the field `final`.
          * **Concealed Dependencies:** Dependencies are not explicit in the constructor, making it harder to know what an object needs just by looking at its signature.
          * **Circular Dependencies:** More prone to causing circular dependency issues at runtime, as Spring might create partial objects to resolve them.
      * **Generally Discouraged** for critical application components.

    <!-- end list -->

    ```java
    package com.example.app.service;

    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.stereotype.Service;

    @Service
    public class ReportingService {
        @Autowired
        private ReportingRepository reportingRepository; // Field Injection
        // This is generally discouraged in production applications for the reasons above.

        public ReportingService() {
            System.out.println("ReportingService: Field Injection (constructor)");
        }
        // ... methods using reportingRepository
    }
    ```

    **Dummy repositories for example:**

    ```java
    package com.example.app.service; // Not a good package for repository, just for demo
    import org.springframework.stereotype.Repository;

    @Repository
    public class ProductRepository { }

    @Repository
    public class InventoryRepository { }

    @Repository
    public class ReportingRepository { }
    ```

### **7.3. When `@Autowired` is optional vs required**

  * **`required = true` (Default):** By default, `@Autowired` dependencies are **required**. If Spring cannot find a matching bean in the context, it will throw a `NoSuchBeanDefinitionException` during application startup.

    ```java
    @Autowired(required = true) // Redundant, as true is default
    private MyRequiredBean requiredBean;
    ```

  * **`required = false`:** You can set `required = false` if a dependency is **optional**. If Spring can't find the bean, it will simply not inject anything (the field will remain `null`, or the setter won't be called). You must handle the `null` case in your code.

    ```java
    @Autowired(required = false)
    private MyOptionalBean optionalBean; // Might be null if no bean is found
    ```

  * **`@Autowired` on a single constructor (Spring 4.3+):** If a class has *only one* constructor, Spring 4.3 and later versions automatically assume it's the constructor to use for dependency injection, even without `@Autowired`. This makes constructor injection even cleaner.

    ```java
    // No @Autowired needed here if this is the only constructor
    public ProductService(ProductRepository productRepository) {
        this.productRepository = productRepository;
    }
    ```

### **7.4. Compare with `@Inject` and manual wiring**

  * **`@Inject` (JSR-330 Standard):**

      * `@Inject` is part of the Java Dependency Injection (JSR-330) standard. It's functionally very similar to `@Autowired`.
      * **Key Difference:** `@Inject` is a standard annotation, meaning it's not Spring-specific. If you want your code to be less coupled to the Spring framework (e.g., for easier migration to another DI framework), `@Inject` is a good choice.
      * **Requires:** You need to include `javax.inject` (or `jakarta.inject` for Jakarta EE 9+) on your classpath.
      * **Functionality:** Supports field, method, and constructor injection, similar to `@Autowired`. It works seamlessly with `@Qualifier`.

    <!-- end list -->

    ```java
    import javax.inject.Inject; // Or jakarta.inject.Inject for newer Spring Boot versions

    @Service
    public class MyOtherService {
        @Inject // Works just like @Autowired
        private SomeDependency someDependency;
    }
    ```

  * **Manual Wiring (Traditional Java or `@Bean` methods):**

      * This is what we saw with `@Configuration` and `@Bean` methods where you explicitly create instances using `new` and pass dependencies in the constructor or via setters.
      * **Pros:** Complete control, no reliance on annotation scanning.
      * **Cons:** Can be verbose, boilerplate, and harder to manage for many beans.
      * **Use Case:** Primarily for third-party libraries or very complex, conditional bean creation where declarative `@Autowired` isn't flexible enough.

    <!-- end list -->

    ```java
    // From @Configuration + @Bean example:
    @Bean
    public MyMessageSender myMessageSender() {
        // Manual wiring, passing the dependency explicitly
        return new MyMessageSender(emailService());
    }
    ```

**Diagram: `@Autowired` Mechanism**

```
+-----------------------------------+
|      Spring IoC Container         |
|      (holds all registered beans) |
+-----------------------------------+
        |
        |  When creating Bean A:
        V
+-----------------------------------+
|      Bean A Instantiation         |
+-----------------------------------+
        | Looks for:
        | - Constructor with @Autowired
        | - Fields with @Autowired
        | - Setters with @Autowired
        V
+-----------------------------------+
|      Dependency Resolution        |
|      (e.g., needs Bean B)         |
+-----------------------------------+
        |
        |  Container searches for Bean B:
        |  - Is there a unique Bean B? -> Inject
        |  - Are there multiple Bean B's?
        |    - Is one @Primary? -> Inject @Primary B
        |    - Is @Qualifier used? -> Inject B matching qualifier
        |    - Else -> Error (NoUniqueBeanDefinitionException)
        V
+-----------------------------------+
|      Inject Dependency (Bean B)   |
|      into Bean A                  |
+-----------------------------------+
        |
        V
+-----------------------------------+
|      Bean A is Ready for Use      |
+-----------------------------------+
```

**Interview Questions & Answers (`@Autowired`):**

1.  **Q:** What is the purpose of `@Autowired`?
      * **A:** `@Autowired` is a Spring annotation used for automatic dependency injection. It tells Spring to find a suitable bean in the application context and inject it into the annotated field, constructor, or setter method.
2.  **Q:** Which type of dependency injection is generally recommended in Spring, and why?
      * **A:** Constructor injection is generally recommended. It promotes immutability of dependencies, makes the class easier to test independently of the Spring container, and ensures that all mandatory dependencies are provided at object creation, leading to a valid object state.
3.  **Q:** What's the difference between `@Autowired` and `@Inject`?
      * **A:** Functionally, they are very similar. The main difference is that `@Autowired` is a Spring-specific annotation, whereas `@Inject` is part of the Java Dependency Injection (JSR-330) standard. Using `@Inject` can make your code less coupled to the Spring framework.
4.  **Q:** What happens if Spring cannot find a bean for an `@Autowired` dependency? How can you make a dependency optional?
      * **A:** By default (`required = true`), Spring will throw a `NoSuchBeanDefinitionException` during application startup if it cannot find a matching bean. To make a dependency optional, you can set `@Autowired(required = false)`, in which case the field or parameter will remain `null` if the bean is not found.

-----

## **8. `@Value`**

`@Value` is a powerful annotation for injecting values from various sources into Spring beans. This is commonly used for externalizing configuration.

### **8.1. Inject values from `application.properties` or environment variables**

The most common use case is injecting properties defined in `application.properties` (or `application.yml`) files.

**`application.properties`:**

```properties
app.name=My Awesome App
app.version=1.0.0
database.url=jdbc:mysql://localhost:3306/mydb
```

**Java Code:**

```java
package com.example.app.component;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;

@Component
public class AppInfo {

    @Value("${app.name}") // Injects value of 'app.name' property
    private String applicationName;

    @Value("${app.version}")
    private String applicationVersion;

    @Value("${database.url}")
    private String dbUrl;

    @Value("${non.existent.property:defaultValue}") // Using a default value if property is not found
    private String optionalProperty;

    public void printAppInfo() {
        System.out.println("Application Name: " + applicationName);
        System.out.println("Application Version: " + applicationVersion);
        System.out.println("Database URL: " + dbUrl);
        System.out.println("Optional Property (with default): " + optionalProperty);
    }
}
```

**Injecting Environment Variables:**

You can also inject environment variables directly. For instance, if you have an environment variable `JAVA_HOME`:

```java
package com.example.app.component;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;

@Component
public class EnvInfo {

    @Value("${JAVA_HOME}") // Injects the value of the JAVA_HOME environment variable
    private String javaHome;

    public void printEnvInfo() {
        System.out.println("JAVA_HOME: " + javaHome);
    }
}
```

### **8.2. How to use SpEL (Spring Expression Language) inside `@Value`**

`@Value` is incredibly flexible because it supports Spring Expression Language (SpEL). SpEL allows you to write powerful expressions to retrieve values, perform operations, and even access other beans.

**Common SpEL examples with `@Value`:**

  * **String concatenation:**
    ```java
    @Value("${app.name} v${app.version}")
    private String fullAppName; // "My Awesome App v1.0.0"
    ```
  * **Accessing System Properties:**
    ```java
    @Value("#{systemProperties['os.name']}")
    private String osName; // e.g., "Windows 10" or "Linux"
    ```
  * **Accessing other beans (less common, usually better with @Autowired):**
    ```java
    // Assuming a bean named 'emailService' exists
    @Value("#{emailService.getClass().getSimpleName()}")
    private String emailServiceName; // "EmailService"
    ```
  * **Mathematical operations:**
    ```java
    @Value("#{10 + 20}")
    private int sum; // 30
    ```
  * **Boolean expressions:**
    ```java
    @Value("#{${my.feature.enabled:false} ? 'Feature is ON' : 'Feature is OFF'}")
    private String featureStatus;
    // my.feature.enabled=true -> "Feature is ON"
    // my.feature.enabled=false (or missing) -> "Feature is OFF"
    ```

**Example combining properties and SpEL:**

```java
package com.example.app.component;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;

@Component
public class SpelDemo {

    // Inject the app.name property, convert to uppercase
    @Value("${app.name:Unknown}")
    private String appNameFromProp;

    @Value("#{ '${app.name}'.toUpperCase() }") // SpEL to uppercase the property
    private String appNameUpper;

    // Inject system property 'user.home'
    @Value("#{systemProperties['user.home']}")
    private String userHomeDir;

    // A default value in SpEL using Elvis operator (?:)
    @Value("#{ ${server.port:8080} > 1024 ? 'High Port' : 'Low Port' }")
    private String portType;

    public void printSpelInfo() {
        System.out.println("App Name from Prop: " + appNameFromProp);
        System.out.println("App Name Uppercase (via SpEL): " + appNameUpper);
        System.out.println("User Home Directory: " + userHomeDir);
        System.out.println("Port Type: " + portType);
    }
}
```

**Running `@Value` examples:**
Create a `Main` class to run them:

```java
package com.example.app;

import com.example.app.component.AppInfo;
import com.example.app.component.EnvInfo;
import com.example.app.component.SpelDemo;
import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.PropertySource;

@Configuration
@ComponentScan(basePackages = "com.example.app.component")
@PropertySource("classpath:application.properties") // Load properties from this file
public class ValueDemoApplication {

    public static void main(String[] args) {
        ApplicationContext context = new AnnotationConfigApplicationContext(ValueDemoApplication.class);

        AppInfo appInfo = context.getBean(AppInfo.class);
        appInfo.printAppInfo();

        System.out.println("\n--- Environment Info ---");
        EnvInfo envInfo = context.getBean(EnvInfo.class);
        envInfo.printEnvInfo();

        System.out.println("\n--- SpEL Demo ---");
        SpelDemo spelDemo = context.getBean(SpelDemo.class);
        spelDemo.printSpelInfo();

        ((AnnotationConfigApplicationContext) context).close();
    }
}
```

**Sample `application.properties` for the above demo:**

```properties
app.name=MyTestApp
app.version=2.0.0
database.url=jdbc:postgresql://localhost:5432/testdb
non.existent.property=
server.port=8081
```

**Output (will vary based on your system properties and env vars):**

```
Application Name: MyTestApp
Application Version: 2.0.0
Database URL: jdbc:postgresql://localhost:5432/testdb
Optional Property (with default): defaultValue

--- Environment Info ---
JAVA_HOME: C:\Program Files\Java\jdk-17 (or your JAVA_HOME path)

--- SpEL Demo ---
App Name from Prop: MyTestApp
App Name Uppercase (via SpEL): MYTESTAPP
User Home Directory: C:\Users\YourUser (or your home directory)
Port Type: High Port
```

### **8.3. Show real-world example using config values like URL or port**

Imagine a `ThirdPartyApiClient` that needs to connect to an external service. Its base URL and API key might come from configuration.

```java
package com.example.app.client;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;

@Component
public class ThirdPartyApiClient {

    @Value("${api.service.base-url}")
    private String baseUrl;

    @Value("${api.service.api-key}")
    private String apiKey;

    @Value("${api.service.timeout.seconds:30}") // Default to 30 seconds if not specified
    private int timeoutSeconds;

    public void callService(String endpoint) {
        System.out.println("Calling external service:");
        System.out.println("  Base URL: " + baseUrl);
        System.out.println("  API Key: " + apiKey);
        System.out.println("  Timeout: " + timeoutSeconds + " seconds");
        System.out.println("  Endpoint: " + endpoint);
        // Real-world: Use a HttpClient to make the actual call
    }
}
```

**`application.properties` (or `application.yml`):**

```properties
api.service.base-url=https://api.external.com/v1
api.service.api-key=YOUR_SECRET_API_KEY_HERE
# api.service.timeout.seconds=60 // Uncomment to override default
```

**Interview Questions & Answers (`@Value`):**

1.  **Q:** What is `@Value` used for in Spring?
      * **A:** `@Value` is used to inject external configuration values (like properties from `application.properties`, environment variables, or system properties) directly into fields, constructor parameters, or method parameters of Spring-managed beans.
2.  **Q:** How can you provide a default value for a property using `@Value` if it's not found?
      * **A:** You can specify a default value directly within the `@Value` annotation using the colon (`:`) operator, like `@Value("${my.property:defaultValue}")`.
3.  **Q:** What is SpEL, and how is it used with `@Value`?
      * **A:** SpEL (Spring Expression Language) is a powerful expression language that can be used within `@Value` annotations (enclosed in `#{}`). It allows for more complex value injection, such as performing calculations, accessing system properties, manipulating strings, or even referencing other beans.

-----

## **Cheat Sheet: All 10 Annotations**

| Annotation             | Category           | Purpose & Use Cases                                                                                                              | Typical Location           | Default Scope    |
| :--------------------- | :----------------- | :------------------------------------------------------------------------------------------------------------------------------- | :------------------------- | :--------------- |
| `@Component`           | Stereotype         | Generic Spring-managed component. Base for other stereotypes. For general utility classes.                                        | Any Spring-managed class   | `singleton`      |
| `@Service`             | Stereotype         | Business logic layer. Semantic hint for service-oriented classes.                                                                | Service Layer classes      | `singleton`      |
| `@Repository`          | Stereotype         | Data access layer. Enables Spring's exception translation. For DAO/Repository classes.                                          | Data Access Layer classes  | `singleton`      |
| `@Controller`          | Stereotype         | Presentation layer (MVC). Handles web requests, typically returns views.                                                          | Controller classes         | `singleton`      |
| `@Configuration`       | Bean Definition    | Marks a class as a source of bean definitions using `@Bean` methods. Replaces XML config.                                         | Configuration class        | `singleton`      |
| `@Bean`                | Bean Definition    | Marks a method within a `@Configuration` class. Its return value is registered as a Spring bean.                                 | Method within `@Configuration` | `singleton`      |
| `@ComponentScan`       | Bean Discovery     | Scans specified packages for Spring stereotype annotations (`@Component`, `@Service`, etc.) to register them as beans.           | Main App / `@Configuration` | N/A (config)     |
| `@Scope`               | Bean Lifecycle     | Defines the lifecycle and visibility of a bean (e.g., `singleton`, `prototype`, `request`, `session`).                          | Class-level or `@Bean` method | N/A (config)     |
| `@Lazy`                | Bean Lifecycle     | Defers bean instantiation until it's first accessed. Improves startup time, can help with circular dependencies.                 | Class-level or `@Bean` method | N/A (config)     |
| `@Primary`             | Dependency Resolve | Designates a single default bean when multiple beans of the same type exist.                                                     | Class-level or `@Bean` method | N/A (config)     |
| `@Qualifier`           | Dependency Resolve | Specifies a particular bean to inject by its unique name when multiple beans of the same type exist. Works with `@Autowired`.    | Injection point (`@Autowired` target) | N/A (config)     |
| `@Autowired`           | Dependency Inject  | Automatically injects dependencies by type. Can be on constructors, fields, or setters.                                         | Injection point (ctor, field, setter) | N/A (config)     |
| `@Value`               | Dependency Inject  | Injects external configuration values (properties, env vars, SpEL expressions) into fields/parameters.                           | Field, Ctor, Method Param  | N/A (config)     |

-----

## **Comparison Table: `@Bean` vs `@Component`**

| Feature/Aspect      | `@Bean`                                                          | `@Component`                                                 |
| :------------------ | :--------------------------------------------------------------- | :----------------------------------------------------------- |
| **Usage** | Annotation on a method within a `@Configuration` class.          | Annotation on a class.                                       |
| **Control** | You manually instantiate the object within the method. Gives fine-grained control over object creation. | Spring automatically instantiates the object during scanning. |
| **Location** | Always inside a `@Configuration` class.                          | On any class that is a Spring-managed component.             |
| **When to use** | - When configuring third-party classes (where you can't add `@Component`).\<br\>- When you need complex initialization logic.\<br\>- When you want to create multiple instances of the same class with different configurations. | - For your own application classes that Spring should automatically discover and manage.\<br\>- Most common approach for typical Spring beans. |
| **Object Creation** | Explicitly `return new MyObject();`                              | Implicit via Spring's reflection and instantiation.          |
| **Testability** | Bean logic can be tested in isolation (just a method call).      | Requires Spring context for full integration testing.        |
| **Readability** | Can be more verbose for simple beans, but clearer for complex ones. | Concise for simple beans.                                    |

-----

## **Common Mistakes and Misconceptions**

1.  **Using `@Component` in a `@Configuration` class's bean method:**
      * **Mistake:**
        ```java
        @Configuration
        public class MyConfig {
            @Bean
            @Component // DON'T DO THIS! @Component is for classes, @Bean for methods
            public MyService myService() {
                return new MyService();
            }
        }
        ```
      * **Explanation:** `@Component` is a class-level annotation for auto-detection. `@Bean` is a method-level annotation to explicitly define a bean. They serve different purposes and should not be combined on the same bean definition method. The actual `MyService` class can be `@Component` if you want it discovered via scanning, but the `@Bean` method itself doesn't need it.
2.  **Forgetting `@ComponentScan` or incorrect `basePackages`:**
      * **Mistake:** You've annotated classes with `@Service` or `@Repository`, but Spring isn't finding them.
      * **Explanation:** Spring needs to know *where* to look. Ensure your `@SpringBootApplication` is in a parent package of your components, or explicitly configure `basePackages` in `@ComponentScan`.
3.  **Using Field Injection excessively:**
      * **Mistake:** `@Autowired private MyService myService;` for all dependencies.
      * **Explanation:** While convenient, it makes your classes harder to test and hides dependencies. Prefer constructor injection for mandatory dependencies.
4.  **Not understanding `singleton` vs. `prototype` scopes:**
      * **Mistake:** Assuming a bean will always be a new instance (prototype) when it's actually a singleton, leading to state issues.
      * **Explanation:** Remember that `singleton` is the default. If your bean needs to be stateful and unique per usage, explicitly mark it as `@Scope("prototype")`.
5.  **Not resolving multiple bean candidates:**
      * **Mistake:** Having two `MessageService` implementations and `@Autowired MessageService;` without `@Primary` or `@Qualifier`.
      * **Explanation:** Spring needs clear instructions when there's ambiguity. Use `@Primary` for a default choice or `@Qualifier` for explicit selection.
6.  **Misunderstanding `@Lazy` impact:**
      * **Mistake:** Lazy loading a critical, always-needed bean and wondering why the first request is slow.
      * **Explanation:** `@Lazy` speeds up *startup* but shifts the cost of instantiation to the *first access*. Use it judiciously for non-critical or rarely used components.

-----

## **Small Quiz to Test Knowledge**

Take a moment to answer these questions\!

1.  Which annotation is a general-purpose stereotype for any Spring-managed component?
2.  If you have two classes, `FastProcessor` and `SlowProcessor`, both implementing `DataProcessor`, and you want `FastProcessor` to be injected by default, which annotation would you use on `FastProcessor`?
3.  What's the primary benefit of using `constructor injection` over `field injection` for `@Autowired` dependencies?
4.  You have a class `MyDataSource` from a third-party library that you cannot modify. How would you register an instance of `MyDataSource` as a Spring bean?
5.  What's the default scope of a Spring bean, and what does it imply about its lifecycle?
6.  You have a property `my.app.version=1.2.3` in `application.properties`. How would you inject this value into a String field `appVersion` in your Spring bean?
7.  If your Spring Boot application's main class is in `com.example.app`, and you have a `@Service` in `com.example.app.business.logic`, will Spring automatically find it? Why or why not?
8.  When would you consider using `@Lazy` for a bean?

-----

## **Quiz Answers**

1.  `@Component`
2.  `@Primary`
3.  Constructor injection promotes immutability, makes the class easier to test independently (without the Spring container), and ensures all mandatory dependencies are provided at object creation, leading to a valid object state.
4.  You would use a `@Configuration` class and define a `@Bean` method that instantiates and returns `MyDataSource`.
    ```java
    @Configuration
    public class DataSourceConfig {
        @Bean
        public MyDataSource myDataSource() {
            return new MyDataSource(); // Assuming MyDataSource has a default constructor
        }
    }
    ```
5.  The default scope is `singleton`. It implies that only one instance of the bean is created per Spring IoC container, and all requests for that bean will return the same instance. Spring manages its complete lifecycle.
6.  You would use the `@Value` annotation: `@Value("${my.app.version}") private String appVersion;`
7.  Yes, Spring will automatically find it. `@SpringBootApplication` (which includes `@ComponentScan`) by default scans the package of the main application class (`com.example.app`) and all its sub-packages, including `com.example.app.business.logic`.
8.  You would consider using `@Lazy` to improve application startup performance (by deferring instantiation of rarely used or resource-heavy beans) or potentially to help break complex circular dependencies during bean creation.

-----

## **One-Page Visual Summary: Spring Bean Definition & Configuration**

````
+--------------------------------------------------------------------------------------------------------------------------------------------------------------------+
|                                                SPRING BEAN DEFINITION & CONFIGURATION - VISUAL SUMMARY                                                           |
+--------------------------------------------------------------------------------------------------------------------------------------------------------------------+
|                                                                                                                                                                    |
|  [Spring IoC Container] - The heart of Spring. Manages your application's objects (Beans).                                                                         |
|                                                                                                                                                                    |
|  ----------------------------------------------------------------------------------------------------------------------------------------------------------------  |
|  **I. Bean Definition & Discovery** |
|  ----------------------------------------------------------------------------------------------------------------------------------------------------------------  |
|                                                                                                                                                                    |
|  1. **Annotation-based (Auto-discovery):** |
|     `@Component`        Generic Spring-managed bean.                                                                                                             |
|     `@Service`          Business logic layer.                                                                                                                      |
|     `@Repository`       Data access layer (DB interaction, exception translation).                                                                                 |
|     `@Controller`       Web layer (MVC controller). (`@RestController` for REST APIs)                                                                                |
|                                                                                                                                                                    |
|     **`@ComponentScan`**: Tells Spring where to find these annotated classes.                                                                                     |
|        - Default: Scans package of `@SpringBootApplication` and its sub-packages.                                                                                 |
|        - Custom: `basePackages = {"pkg1", "pkg2"}` or `basePackageClasses = {Class1.class}`.                                                                     |
|                                                                                                                                                                    |
|  2. **Java-based (Explicit Definition):** |
|     **`@Configuration`**: Marks a class as a source of bean definitions.                                                                                           |
|     **`@Bean`**: On methods within `@Configuration`. Method return value is a bean. (e.g., for 3rd party libs, complex init)                                      |
|        ```java                                                                                                                                                      |
|        @Configuration                                                                                                                                              |
|        public class AppConfig {                                                                                                                                    |
|            @Bean public MyCustomBean myCustomBean() { return new MyCustomBean(); }                                                                               |
|        }                                                                                                                                                           |
|        ```                                                                                                                                                           |
|                                                                                                                                                                    |
|  ----------------------------------------------------------------------------------------------------------------------------------------------------------------  |
|  **II. Bean Lifecycle & Resolution** |
|  ----------------------------------------------------------------------------------------------------------------------------------------------------------------  |
|                                                                                                                                                                    |
|  **`@Scope`**: Defines how many instances of a bean exist and for how long.                                                                                       |
|     - `singleton` (Default): One instance per container. Shared. (Lifecycle: Managed by Spring)                                                                  |
|     - `prototype`: New instance every time. Not managed by Spring after creation.                                                                                  |
|     - `request`, `session`, `application`, `websocket` (Web-aware scopes)                                                                                          |
|                                                                                                                                                                    |
|  **`@Lazy`**: Defers bean creation until first access.                                                                                                            |
|     - Benefits: Faster startup, can help with circular dependencies.                                                                                               |
|     - Drawback: First access might be slower.                                                                                                                      |
|                                                                                                                                                                    |
|  **Resolving Multiple Beans of Same Type:** |
|     - **`@Primary`**: Designates one bean as the default choice among many.                                                                                        |
|     - **`@Qualifier("beanName")`**: Explicitly specifies which bean to inject by its unique name. (Overrides `@Primary`)                                          |
|                                                                                                                                                                    |
|  ----------------------------------------------------------------------------------------------------------------------------------------------------------------  |
|  **III. Dependency Injection (`@Autowired`) & Value Injection (`@Value`)** |
|  ----------------------------------------------------------------------------------------------------------------------------------------------------------------  |
|                                                                                                                                                                    |
|  **`@Autowired`**: Automatic dependency injection. Spring finds and injects compatible beans.                                                                       |
|     - **Constructor Injection (Recommended):** `final` fields, testable, ensures valid state.                                                                      |
|       ```java @Autowired public MyService(Dependency dep) { ... } // Optional @Autowired for single ctor (Spring 4.3+) ```                                        |
|     - **Setter Injection:** For optional dependencies.                                                                                                             |
|       ```java @Autowired public void setDep(Dependency dep) { ... } ```                                                                                           |
|     - **Field Injection (Discouraged):** Simple but less testable, hides dependencies.                                                                             |
|       ```java @Autowired private Dependency dep; ```                                                                                                              |
|     - `@Inject`: Standard Java alternative to `@Autowired`.                                                                                                        |
|                                                                                                                                                                    |
|  **`@Value("${property.key:defaultValue}")`**: Injects values from configuration (e.g., `application.properties`), env vars, or SpEL.                             |
|     - SpEL (`#{ }`): Allows expressions for dynamic values (`#{systemProperties['os.name']}`, `#{10+20}`).                                                         |
|                                                                                                                                                                    |
+--------------------------------------------------------------------------------------------------------------------------------------------------------------------+
````
