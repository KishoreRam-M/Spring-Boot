### **Mastering Spring Profiles and Configuration**

-----

### **`spring.profiles.active`**

**Concept Overview**

`spring.profiles.active` is a core property that specifies which profiles should be considered "active" for the Spring application. It's the primary mechanism for switching between different environment-specific configurations without changing code. Spring Boot loads properties from `application.properties` (or `application.yml`) by default. When you activate a profile, for example `dev`, Spring also loads `application-dev.properties` and merges it with the default properties. Profile-specific properties **override** default ones.

Spring resolves active profiles in a specific order of precedence, from highest to lowest:

1.  **Command-line arguments** (e.g., `--spring.profiles.active=prod`).
2.  **Environment variables** (e.g., `SPRING_PROFILES_ACTIVE=prod`).
3.  **Servlet context parameters** (for web apps).
4.  **`application.properties`** or `application.yml` file.

**Code Example**

To activate the `dev` profile, you can simply add the following to `application.properties`:

```properties
# src/main/resources/application.properties
spring.profiles.active=dev
```

You would then have a separate file for the `dev` environment:

```properties
# src/main/resources/application-dev.properties
spring.datasource.url=jdbc:h2:mem:dev_db
```

**Real-world Analogy**

Think of `spring.profiles.active` as a set of blueprints for a building. The default blueprint (`application.properties`) is for a generic house. When you specify `spring.profiles.active=prod`, you're grabbing a specialized "production" blueprint (`application-prod.properties`) that modifies the generic one, perhaps adding solar panels and a more robust foundation for a specific location.

**Interview Tip**

  * **Q**: What happens if a property is defined in `application.properties` and also in `application-dev.properties`?
  * **A**: The value in `application-dev.properties` will override the one in `application.properties`. This allows you to define a common configuration in the default file and then only override specific properties for each profile.

**Common Mistake / Pitfall**

Forgetting that command-line arguments have the highest precedence. A common debugging issue is a developer struggling with a profile that won't activate, only to find it's being overridden by an unexpected environment variable or command-line flag.

**Hands-on Task**

Create a `MessageService` bean that reads a `welcome.message` property. In `application.properties`, set a default message. Then, create `application-dev.properties` and `application-prod.properties` with different messages. Run the application with and without activating a profile to observe the different messages.

-----

### **`@Profile` Annotation**

**Concept Overview**

The `@Profile` annotation allows you to conditionally register beans or configuration classes. You can apply it to `@Component`, `@Service`, `@Repository`, or `@Configuration` classes, as well as on `@Bean` methods within a configuration class. A bean or configuration is only created and added to the application context if the specified profile is active. This is a powerful way to provide different implementations of the same service for different environments.

**Code Example**

```java
import org.springframework.context.annotation.Profile;
import org.springframework.stereotype.Service;

public interface NotificationService {
    void sendNotification(String message);
}

@Service
@Profile("dev")
public class DevNotificationService implements NotificationService {
    @Override
    public void sendNotification(String message) {
        System.out.println("DEV: Sending a notification via a mock service: " + message);
    }
}

@Service
@Profile("prod")
public class ProdNotificationService implements NotificationService {
    @Override
    public void sendNotification(String message) {
        // Real-world implementation using an email service, etc.
        System.out.println("PROD: Sending a real notification: " + message);
    }
}
```

**Real-world Analogy**

`@Profile` is like having different sets of tools for a job. For the "development" profile, you use a set of mock tools that are fast and safe to experiment with. For the "production" profile, you use the real, more powerful tools that are ready for the final task.

**Interview Tip**

  * **Q**: What happens if you have two beans of the same type (`NotificationService`) but neither has an active `@Profile`?
  * **A**: Spring will throw a `NoUniqueBeanDefinitionException` because it can't decide which bean to inject. You must ensure that only one bean for a given dependency is active at any time.

**Common Mistake / Pitfall**

Forgetting to specify a profile for a bean, leading to it being loaded regardless of the environment. This can cause unexpected behavior, like a mock service running in production.

**Hands-on Task**

Implement the `NotificationService` example above. Write a `Runner` class that injects `NotificationService` and calls its `sendNotification` method. Run the application with `dev` and `prod` profiles active and observe the different outputs.

-----

### **`@ConfigurationProperties`**

**Concept Overview**

`@ConfigurationProperties` is a powerful way to bind an entire class to a set of properties from configuration files. Instead of using `@Value` on individual fields, you define a single class with fields that correspond to properties with a common prefix. This approach provides several advantages:

  * **Type Safety**: Spring automatically handles type conversion, so you can have properties of different types (e.g., `int`, `boolean`, `List`).
  * **Nested Properties**: You can easily bind nested properties to a class with nested objects.
  * **Validation**: You can use JSR-303 annotations like `@NotNull` or `@Min` to validate the properties.

**Code Example**

`application.yml`:

```yaml
app.security.enabled: true
app.security.roles: ADMIN, USER
app.security.token.expiry: 3600
```

`SecurityProperties.java`:

```java
import jakarta.validation.constraints.Min;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.stereotype.Component;
import java.util.List;

@Component
@ConfigurationProperties(prefix = "app.security")
public class SecurityProperties {
    private boolean enabled;
    private List<String> roles;
    private Token token;

    // Getters and setters omitted for brevity

    public static class Token {
        @Min(value = 1800)
        private int expiry;
        // Getter and setter omitted
    }
}
```

**Real-world Analogy**

Using `@Value` is like manually filling out a form, field by field. `@ConfigurationProperties` is like using a template to fill out the form automatically, where the template knows which properties go into which fields.

**Interview Tip**

  * **Q**: What's the main advantage of `@ConfigurationProperties` over `@Value` for a large configuration?
  * **A**: Readability and maintainability. It centralizes all related properties in a single, type-safe class, making it easy to see the full configuration structure. It also allows for property validation and IDE-based auto-completion.

**Common Mistake / Pitfall**

Forgetting to add `@EnableConfigurationProperties` (or `@SpringBootConfiguration`) to a configuration class, or not annotating the class with a component stereotype like `@Component`. This prevents Spring from discovering and binding the class.

**Hands-on Task**

Create a `ThreadPoolProperties` class with properties for `corePoolSize`, `maxPoolSize`, and `queueCapacity`. Use `@ConfigurationProperties` to bind these properties from `application.yml` and add a validation annotation (`@Min`) to `corePoolSize`.

-----

### **Advanced Insights**

  * **Combining Profiles with `@ConfigurationProperties`**: You can use `@ConfigurationProperties` with profile-specific files. The binding logic will automatically use the values from the active profile's properties.
  * **Dynamic Profile Switching**: While profile switching is typically done at startup, in rare cases (e.g., in a cloud environment), you might need to change a profile. Spring Cloud Config and other tools can help with this, but it's an advanced topic.
  * **Ordering of Profile-Based Beans**: When multiple beans of the same type are available across different profiles, the active profile bean is used. If a bean has no `@Profile` annotation, it's considered to be in the "default" profile and will be loaded unless a specific profile-based bean for that type is active.
  * **Best Practices**:
      * Use `application.yml` for its structured and readable format.
      * Separate configuration for each environment into its own `application-{profile}.yml` file.
      * Use `@ConfigurationProperties` for all custom application settings to get the benefits of type-safety and validation.

**Interview Preparation**

  * **Q**: You have a `DataSource` bean for the `dev` profile and another for the `prod` profile. What happens if you run the application with **no** active profile?

  * **A**: Neither bean will be created. The application will fail to start because a `DataSource` is required but no active profile provides one.

  * **Q**: How can you define a default bean that's used when no specific profile is active?

  * **A**: You can use `@Profile("default")` on the bean definition. This bean will be created when no other profile is explicitly activated.

  * **Q**: What is the difference between `@Profile("dev")` and `@Profile("!prod")`?

  * **A**: `@Profile("dev")` activates the bean only when the `dev` profile is active. `@Profile("!prod")` activates the bean when the `prod` profile is **not** active. This can be useful for fallback configurations.
