### **Mastering Spring Core Utilities**

-----

### **ApplicationContextAware & BeanNameAware**

These interfaces are part of Spring's `Aware` family, allowing beans to "be aware" of the Spring container. They provide a way for a bean to interact with the Spring context itself, which is generally discouraged but necessary in specific scenarios.

**Concept Overview**

  * **`ApplicationContextAware`**: A bean implementing this interface gains access to the entire `ApplicationContext`. Spring calls the `setApplicationContext(ApplicationContext context)` method and injects the context. This allows a bean to programmatically retrieve other beans, publish events, or access resource files.
  * **`BeanNameAware`**: A bean implementing this interface receives its own bean name. Spring calls the `setBeanName(String name)` method after the bean has been instantiated. This is useful for logging, creating dynamic IDs, or registering a bean with a non-Spring registry.

**Code Example**

```java
import org.springframework.beans.factory.BeanNameAware;
import org.springframework.context.ApplicationContext;
import org.springframework.context.ApplicationContextAware;
import org.springframework.stereotype.Component;

@Component
public class MyAwareBean implements ApplicationContextAware, BeanNameAware {

    private String beanName;
    private ApplicationContext context;

    @Override
    public void setApplicationContext(ApplicationContext applicationContext) {
        this.context = applicationContext;
        System.out.println("ApplicationContextAware: Context set. I can now access beans programmatically.");
    }

    @Override
    public void setBeanName(String name) {
        this.beanName = name;
        System.out.println("BeanNameAware: My bean name is '" + this.beanName + "'.");
    }

    public void doSomething() {
        System.out.println("Executing from bean: " + this.beanName);
        // Example of using the context to get another bean
        MyOtherBean otherBean = context.getBean(MyOtherBean.class);
        otherBean.someMethod();
    }
}
```

**Real-world Analogy**

Think of a new employee (`BeanNameAware`) being introduced to the company (`ApplicationContext`). First, they are told their official title (`setBeanName`). Then, they are given a full directory of all employees and departments (`setApplicationContext`), which they can use to find and communicate with anyone in the company.

**Interview Tip**

  * **Q**: Why is using `ApplicationContextAware` often considered a bad practice?
  * **A**: It couples your code tightly to the Spring framework. The bean loses its independence and cannot be tested or used outside of a Spring context. A better practice is to use dependency injection (`@Autowired`) to get specific dependencies rather than the entire context.

**Common Mistake**

Trying to use the injected context or bean name in the constructor. The `Aware` callbacks are invoked *after* the bean is constructed and its properties have been set.

**Hands-on Task**

Create two beans, `ServiceA` and `ServiceB`. Make `ServiceA` implement `ApplicationContextAware`. In `ServiceA`, use the injected `ApplicationContext` to programmatically get an instance of `ServiceB` and call one of its methods.

-----

### **@PostConstruct & @PreDestroy**

These are lifecycle callbacks that give you control over a bean's initialization and destruction. They are part of the JSR-250 specification, making them a standard Java annotation, not just a Spring-specific one.

**Concept Overview**

  * **`@PostConstruct`**: This annotation marks a method to be executed immediately after the bean is constructed and its dependencies have been injected. It's the ideal place for initialization logic that requires all dependencies to be ready.
  * **`@PreDestroy`**: This annotation marks a method to be executed just before the bean is destroyed by the container. It's perfect for cleanup tasks like closing database connections, releasing resources, or deregistering from a registry.

**Code Example**

```java
import jakarta.annotation.PostConstruct;
import jakarta.annotation.PreDestroy;
import org.springframework.stereotype.Component;

@Component
public class LifecycleBean {

    private String resource;

    // This method runs after the bean is created and properties are set
    @PostConstruct
    public void initialize() {
        this.resource = "Resource created";
        System.out.println("@PostConstruct: Resource has been initialized.");
    }

    public void doWork() {
        System.out.println("Doing work with " + this.resource);
    }

    // This method runs before the bean is destroyed
    @PreDestroy
    public void cleanup() {
        this.resource = null;
        System.out.println("@PreDestroy: Resource has been cleaned up.");
    }
}
```

**Real-world Analogy**

`@PostConstruct` is like a factory worker performing a final inspection and setup of a new product on the assembly line before it is shipped out. `@PreDestroy` is like a decommissioning process, where the factory dismantles a machine, carefully turning it off and clearing its parts.

**Interview Tip**

  * **Q**: What is the order of execution for bean lifecycle events?
  * **A**: The order is as follows: 1. Constructor -\> 2. `Aware` interfaces (`BeanNameAware`, etc.) -\> 3. `BeanFactoryPostProcessor` -\> 4. `BeanPostProcessor` (`postProcessBeforeInitialization`) -\> 5. `@PostConstruct` -\> 6. `InitializingBean` (`afterPropertiesSet`) -\> 7. `BeanPostProcessor` (`postProcessAfterInitialization`) -\> **Bean is ready to use**. For destruction: `DisposableBean` -\> `@PreDestroy`.

**Common Mistake**

Using `@PostConstruct` for business logic instead of initialization logic. The method should not have any parameters and should not throw exceptions that are not caught.

**Hands-on Task**

Create a `DatabaseConnectionPool` bean. Use `@PostConstruct` to establish a set of initial connections and `@PreDestroy` to gracefully close them.

-----

### **FactoryBean**

A `FactoryBean` is a special bean that acts as a factory for other beans. Instead of the `FactoryBean` itself being the final product, it produces a different object when requested from the container.

**Concept Overview**

A `FactoryBean` provides a way to customize the bean creation logic. It's useful when you need to create a complex object, a proxy, or a third-party object that Spring doesn't know how to instantiate directly. When a bean with the ID `myFactoryBean` is requested, Spring calls the `getObject()` method of the `FactoryBean` and returns the resulting object instead of the `FactoryBean` instance itself.

**Code Example**

```java
import org.springframework.beans.factory.FactoryBean;
import org.springframework.stereotype.Component;

@Component
public class EncryptionServiceFactory implements FactoryBean<EncryptionService> {

    @Override
    public EncryptionService getObject() throws Exception {
        // Complex object creation logic goes here
        System.out.println("FactoryBean: Creating EncryptionService instance.");
        return new EncryptionService("AES", 256);
    }

    @Override
    public Class<?> getObjectType() {
        return EncryptionService.class;
    }

    @Override
    public boolean isSingleton() {
        return true;
    }
}
```

**Real-world Analogy**

A `FactoryBean` is like a vending machine. When you ask for a "soda" (the bean name), you don't get the vending machine itself. Instead, the machine (`FactoryBean`) internally does some work (takes your money, dispenses a can) and gives you the actual soda (`getObject()`).

**Interview Tip**

  * **Q**: How do you get the `FactoryBean` instance itself instead of the object it produces?
  * **A**: You can prepend an ampersand (`&`) to the bean ID. For example, `context.getBean("&myFactoryBean")` would return the `FactoryBean` instance.

**Common Mistake**

Confusing `FactoryBean` with `BeanFactory`. `FactoryBean` is a bean that *creates* other beans. `BeanFactory` is the core Spring container itself.

**Hands-on Task**

Create a `DataSourceFactoryBean` that produces a `DataSource` object. The factory bean should read database properties from a file and use them to configure a `HikariDataSource`.

-----

### **@Profile**

The `@Profile` annotation allows you to register beans conditionally based on the active Spring profile. This is essential for managing configurations across different environments (e.g., development, testing, production) without changing the code.

**Concept Overview**

You can use `@Profile("dev")` on a component, configuration class, or bean definition method to ensure it's only loaded when the "dev" profile is active. Multiple profiles can be active at once.

**Code Example**

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Profile;

@Configuration
public class DataSourceConfig {

    @Bean
    @Profile("dev")
    public DataSource devDataSource() {
        System.out.println("Loading DEV data source.");
        return new DevDataSource();
    }

    @Bean
    @Profile("prod")
    public DataSource prodDataSource() {
        System.out.println("Loading PROD data source.");
        return new ProdDataSource();
    }
}
```

**Real-world Analogy**

Profiles are like different sets of keys for a building. In "development" mode, you use the developer's key, which gives you access to a sandbox environment. In "production" mode, you use the master key, which gives you access to the live, secure environment. The building remains the same, but the active key determines your access.

**Interview Tip**

  * **Q**: What happens if no profile is active?
  * **A**: If no profile is explicitly active, Spring activates a default profile. You can also specify beans for this default profile using `@Profile("default")`.

**Common Mistake**

Defining multiple beans of the same type for different profiles without a default. If a bean is required and no active profile provides it, the application will fail to start with a `NoUniqueBeanDefinitionException`.

**Hands-on Task**

Create two `MessageService` implementations, `DevMessageService` and `ProdMessageService`, each with a different `@Profile`. In your `application.properties`, activate the `dev` profile using `spring.profiles.active=dev` and observe which bean is loaded.
