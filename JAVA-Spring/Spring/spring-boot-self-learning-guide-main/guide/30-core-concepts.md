# Spring Boot Core Concepts

## Table of Contents
- [Bean Configuration Examples](#bean-configuration-examples)
- [Spring Component Scan](#spring-component-scan)

- A Spring bean is a Java object that is managed by the Spring IoC container.
- The Spring IoC container is responsible for creating, configuring, and managing the lifecycle of Spring beans.
- You can use the generic `@Component` annotation to mark a class as a Spring bean.
- But there are more specific annotations that you can use based on the type of bean you want to create.
  - `@Service` for business logic 
  - `@Repository` for data access
  - `@Controller` for web controllers
  - `@RestController` for REST controllers
- You can also use the `@Bean` annotation to register a bean definition with the container programmatically.
- You can also use the `@Configuration` annotation to mark a class as a Spring configuration class.
- You can inject one bean into another bean using the `@Autowired` annotation.
- You can use constructor injection, setter injection, and field injection.
- It is recommended to use constructor injection for all beans.
- If there is only one constructor for a class, you don't need to annotate the constructor with `@Autowired`.

## Bean Configuration Examples

Annotation based dependency injection example:

```java
@Repository
class UserRepository {
    // data access logic methods
}

@Service
class UserService {
    private final UserRepository userRepository;
    
    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
    
    // business logic methods
}

@Component
class UserMapper {
    // data mapping logic methods
}

@RestController
class UserController {
    private final UserService userService;
    private final UserMapper userMapper;
    
    public UserController(UserService userService, UserMapper userMapper) {
        this.userService = userService;
        this.userMapper = userMapper;
    }
    
    // request handler methods
}
```

Registering a beans using Java Configuration:

```java
@Configuration
class AppConfig {
    @Bean
    public UserReposiotry userReposiotry() {
        return new UserReposiotry();
    }
    
    @Bean
    public UserService userService(UserReposiotry userReposiotry) {
        return new UserService(userReposiotry);
    }
    
    @Bean
    public UserMapper userMapper() {
        return new UserMapper();
    }
}
```

**NOTE:** Usually Java Configuration is used to configure beans of types from external libraries 
as we can't annotate them with Spring annotations.

## Spring Component Scan
You can use the `@ComponentScan` annotation to configure the package(s) where Spring should look for components.

```java
@Configuration
@ComponentScan("com.example.app")
class AppConfig {
    
}
```

In Spring Boot applications, the main entry point is the class with `@SpringBootApplication` annotation.

```java
package com.example.app;

@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

The `@SpringBootApplication` annotation is a meta-annotated with `@Configuration` and `@ComponentScan` annotations.
So, all the components from the `com.example.app` package and its sub-packages will be registered with the Spring IoC container.

**IMPORTANT:** You should keep all the classes in the same package or sub-packages of the main entry point class.
Otherwise, Spring will not be able to find them by default.

---
[Home](../README.md) | [← Previous: Getting Started](20-getting-started.md) | [Next: Spring MVC REST APIs →](40-spring-mvc-rest-apis.md)
