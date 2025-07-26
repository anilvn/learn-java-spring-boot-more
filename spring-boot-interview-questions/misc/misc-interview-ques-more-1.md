# Spring Boot Interview Questions & Answers - Comprehensive Notes

## 1. CommandLineRunner and ApplicationRunner

### Purpose
Both interfaces are used to execute code **after the Spring application context has been initialized** (right after startup, before the application starts serving requests).

### Key Differences
- **CommandLineRunner**: Has a single method `run(String... args)` which receives application arguments as a simple string array
- **ApplicationRunner**: Has `run(ApplicationArguments args)` method that provides richer access to parsed arguments

### Use Cases
- Initializing test data
- Running startup checks
- Logging configuration
- Database seeding

### Example
```java
@Component
public class StartupRunner implements CommandLineRunner {
    @Override
    public void run(String... args) {
        System.out.println("Application started with args: " + Arrays.toString(args));
    }
}
```

---

## 2. Configuration and @Configuration

### @Configuration
- Marks a class as a **source of bean definitions**
- Similar to XML configuration files
- Used with `@Bean` methods to define beans

### Example
```java
@Configuration
public class AppConfig {
    @Bean
    public MyService myService() {
        return new MyServiceImpl();
    }
}
```

### @ConfigurationProperties
- Maps external properties (from application.properties or YAML) into a POJO
- Useful for type-safe configuration

```java
@Component
@ConfigurationProperties(prefix = "app")
public class AppProperties {
    private String name;
    private String version;
    // getters and setters
}
```

---

## 3. Overriding Default Auto-Configurations

### Methods to Override
1. **Define your own @Bean** of the same type and name
2. **Exclude specific auto-configuration classes**
3. **Use conditional annotations** like `@ConditionalOnMissingBean`

### Examples

#### Excluding Auto-Configuration
```java
@SpringBootApplication(exclude = {DataSourceAutoConfiguration.class})
public class Application {
    // ...
}
```

#### Custom Bean Definition
```java
@Configuration
public class CustomDataSourceConfig {
    @Bean
    public DataSource dataSource() {
        // define and return your own DataSource
        return new HikariDataSource();
    }
}
```

---

## 4. API Versioning in Spring Boot

### Versioning Strategies

#### 1. URI Versioning
```java
@RestController
@RequestMapping("/api/v1/customers")
public class CustomerControllerV1 {
    // V1 endpoints
}

@RestController
@RequestMapping("/api/v2/customers")
public class CustomerControllerV2 {
    // V2 endpoints
}
```

#### 2. Request Parameter Versioning
- `/customers?version=1`

#### 3. Header Versioning
- Custom header: `X-API-VERSION: 1`

#### 4. Content Negotiation
- Accept header: `Accept: application/vnd.example.v1+json`

---

## 5. Custom Annotations Implementation

### Creating Custom Annotations
Use `@interface` to create custom annotations, optionally with Spring AOP for behavior.

### Example: Custom Logging Annotation
```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface LogExecutionTime {
}
```

### AOP Implementation
```java
@Aspect
@Component
public class LoggingAspect {
    @Around("@annotation(LogExecutionTime)")
    public Object logTime(ProceedingJoinPoint joinPoint) throws Throwable {
        long start = System.currentTimeMillis();
        Object proceed = joinPoint.proceed();
        long duration = System.currentTimeMillis() - start;
        System.out.println(joinPoint.getSignature() + " executed in " + duration + "ms");
        return proceed;
    }
}
```

---

## 6. Multiple SQL Databases Configuration

### Problem
Spring Boot's default `spring.datasource.*` supports only one datasource. For multiple SQL databases, explicit configuration is required.

### Solution Steps

#### 1. Application Configuration
```yaml
spring:
  datasource:
    primary:
      url: jdbc:postgresql://localhost:5432/primarydb
      username: postgres
      password: postgres
    secondary:
      url: jdbc:postgresql://localhost:5432/secondarydb
      username: postgres
      password: postgres
```

#### 2. Primary Database Configuration
```java
@Configuration
@EnableJpaRepositories(
    basePackages = "com.example.primary.repository",
    entityManagerFactoryRef = "primaryEntityManagerFactory",
    transactionManagerRef = "primaryTransactionManager"
)
public class PrimaryDbConfig {
    @Bean
    @Primary
    @ConfigurationProperties("spring.datasource.primary")
    public DataSource primaryDataSource() {
        return DataSourceBuilder.create().build();
    }

    @Bean
    @Primary
    public LocalContainerEntityManagerFactoryBean primaryEntityManagerFactory(
            EntityManagerFactoryBuilder builder) {
        return builder
            .dataSource(primaryDataSource())
            .packages("com.example.primary.entity")
            .persistenceUnit("primary")
            .build();
    }

    @Bean
    @Primary
    public PlatformTransactionManager primaryTransactionManager(
            @Qualifier("primaryEntityManagerFactory") EntityManagerFactory entityManagerFactory) {
        return new JpaTransactionManager(entityManagerFactory);
    }
}
```

#### 3. Secondary Database Configuration
```java
@Configuration
@EnableJpaRepositories(
    basePackages = "com.example.secondary.repository",
    entityManagerFactoryRef = "secondaryEntityManagerFactory",
    transactionManagerRef = "secondaryTransactionManager"
)
public class SecondaryDbConfig {
    @Bean
    @ConfigurationProperties("spring.datasource.secondary")
    public DataSource secondaryDataSource() {
        return DataSourceBuilder.create().build();
    }

    @Bean
    public LocalContainerEntityManagerFactoryBean secondaryEntityManagerFactory(
            EntityManagerFactoryBuilder builder) {
        return builder
            .dataSource(secondaryDataSource())
            .packages("com.example.secondary.entity")
            .persistenceUnit("secondary")
            .build();
    }

    @Bean
    public PlatformTransactionManager secondaryTransactionManager(
            @Qualifier("secondaryEntityManagerFactory") EntityManagerFactory entityManagerFactory) {
        return new JpaTransactionManager(entityManagerFactory);
    }
}
```

### Key Points
- Each datasource is fully isolated with its own DataSource, EntityManagerFactory, and TransactionManager
- Use `@EnableJpaRepositories` to specify which repositories belong to which datasource
- Use `@Primary` to mark the default datasource

---

## 7. Spring Security - Custom Login/Logout Flows

### Customization Options
- Custom login page
- Custom logout URL or success URL
- Custom success/failure handlers

### Security Configuration Example
```java
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
            .formLogin()
                .loginPage("/custom-login")         // your custom login page URL
                .loginProcessingUrl("/perform-login") // URL Spring uses to process the login
                .defaultSuccessUrl("/home", true)   // redirect on success
                .failureUrl("/custom-login?error")  // redirect on failure
            .and()
            .logout()
                .logoutUrl("/perform-logout")       // URL to trigger logout
                .logoutSuccessUrl("/custom-login?logout"); // redirect after logout
    }
}
```

### Custom Handlers
You can implement `AuthenticationSuccessHandler` and `AuthenticationFailureHandler` for complex logic.

---

## 8. Role-Based Access Control in Spring Security

### Configuration-Based Authorization
```java
@Override
protected void configure(HttpSecurity http) throws Exception {
    http
        .authorizeRequests()
            .antMatchers("/admin/**").hasRole("ADMIN")
            .antMatchers("/user/**").hasAnyRole("USER", "ADMIN")
            .antMatchers("/", "/public/**").permitAll()
            .anyRequest().authenticated();
}
```

### Method-Level Security
```java
@Service
public class OrderService {
    @PreAuthorize("hasRole('ADMIN')")
    public void deleteOrder(Long id) {
        // only admin can delete
    }

    @Secured({"ROLE_USER", "ROLE_ADMIN"})
    public List<Order> getOrders() {
        return orders;
    }
}
```

### Enable Method Security
```java
@EnableGlobalMethodSecurity(prePostEnabled = true, securedEnabled = true)
```

**Important**: Spring automatically prefixes roles with `ROLE_`. So `hasRole("ADMIN")` expects the authority `ROLE_ADMIN`.

---

## 9. Connecting to Different Database Types (PostgreSQL + MongoDB)

### Overview
Spring Boot supports **multiple data sources** out of the box. Because PostgreSQL is **relational** and MongoDB is **NoSQL**, the setup is quite natural — each uses its own starter, configuration, and repositories.

### Step-by-Step Implementation

#### 1️⃣ Add Dependencies in `pom.xml` (or Gradle)
```xml
<!-- PostgreSQL -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
    <groupId>org.postgresql</groupId>
    <artifactId>postgresql</artifactId>
</dependency>

<!-- MongoDB -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-mongodb</artifactId>
</dependency>
```

#### 2️⃣ Configure PostgreSQL Datasource in `application.yml`
```yaml
spring:
  datasource:
    url: jdbc:postgresql://localhost:5432/mydb
    username: postgres
    password: postgres
  jpa:
    hibernate:
      ddl-auto: update
    show-sql: true
```

#### 3️⃣ Configure MongoDB Connection
```yaml
spring:
  data:
    mongodb:
      uri: mongodb://localhost:27017/my_mongo_db
```

#### 4️⃣ Use Repositories
For **PostgreSQL**: Create entities and extend `JpaRepository`
For **MongoDB**: Create documents (`@Document`) and extend `MongoRepository`

### Repository Examples

#### PostgreSQL Entity and Repository
```java
@Entity
public class User {
    @Id
    private Long id;
    private String name;
    // getters and setters
}

@Repository
public interface UserRepository extends JpaRepository<User, Long> { }
```

#### MongoDB Document and Repository
```java
@Document(collection = "products")
public class Product {
    @Id
    private String id;
    private String name;
    // getters and setters
}

@Repository
public interface ProductRepository extends MongoRepository<Product, String> { }
```

### Key Points
- Spring Boot makes this easy because **JPA** and **MongoDB** use different configurations and infrastructure
- Both can run in the **same application context** without conflict
- Use `JpaRepository` for SQL databases and `MongoRepository` for MongoDB
- If you want **two SQL databases** (e.g., two PostgreSQL DBs), then you'd need:
  - Two `DataSource` beans
  - Two `EntityManagerFactory` beans
  - Use `@EnableJpaRepositories` with `basePackages` pointing to each repository package

### Interview Summary
*"In Spring Boot, you can connect to different types of databases (e.g., PostgreSQL and MongoDB) by adding their respective starters and configuring each data source separately in application.yml. You then use separate repositories: JpaRepository for SQL and MongoRepository for MongoDB. Spring Boot automatically wires everything based on the starters and configuration."*

---

## Interview Tips

1. **Be concise but comprehensive** - Explain the concept, show the code, mention use cases
2. **Always provide practical examples** - Interviewers appreciate real-world scenarios
3. **Mention best practices** - Such as using `@Primary` for default beans, proper package organization
4. **Understand the "why"** - Know when to use each approach and their trade-offs
5. **Be ready for follow-up questions** - Such as performance implications, error handling, testing strategies