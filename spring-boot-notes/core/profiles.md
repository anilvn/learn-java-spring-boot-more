In **Spring Boot**, **profiles** are used to define different configurations for different environments (like `dev`, `test`, `prod`, etc.). You can isolate configuration settings and beans specific to a profile. Here's a step-by-step guide on how to **create and use various profiles in Spring Boot**.

---

## ✅ Step 1: Define Profile-Specific `application` Files

Create different configuration files for each profile in `src/main/resources`:

```
application.properties            <-- Common config
application-dev.properties        <-- Dev profile
application-test.properties       <-- Test profile
application-prod.properties       <-- Prod profile
```

---

## ✅ Step 2: Add Profile-Specific Properties

**application-dev.properties**

```properties
server.port=8081
spring.datasource.url=jdbc:mysql://localhost:3306/devdb
logging.level.root=DEBUG
```

**application-prod.properties**

```properties
server.port=8080
spring.datasource.url=jdbc:mysql://localhost:3306/proddb
logging.level.root=INFO
```

---

## ✅ Step 3: Activate a Profile

You can activate a profile using one of the following ways:

### ✅ Option 1: Set in `application.properties`

```properties
spring.profiles.active=dev
```

### ✅ Option 2: Pass as JVM argument

```bash
-Dspring.profiles.active=prod
```

### ✅ Option 3: Set via Command Line

```bash
java -jar myapp.jar --spring.profiles.active=prod
```

### ✅ Option 4: Set in `application.yml`

```yaml
spring:
  profiles:
    active: test
```

---

## ✅ Step 4: Use `@Profile` Annotation in Code

You can conditionally load beans based on the profile:

```java
@Component
@Profile("dev")
public class DevDataSourceConfig implements DataSourceConfig {
    // dev-specific beans
}
```

```java
@Component
@Profile("prod")
public class ProdDataSourceConfig implements DataSourceConfig {
    // prod-specific beans
}
```

---

## ✅ Step 5: Conditional Beans in Configuration Class

```java
@Configuration
public class AppConfig {

    @Bean
    @Profile("dev")
    public DataSource devDataSource() {
        return new HikariDataSource(); // dev setup
    }

    @Bean
    @Profile("prod")
    public DataSource prodDataSource() {
        return new HikariDataSource(); // prod setup
    }
}
```

---

## ✅ Step 6: Use `@Value` or `@ConfigurationProperties` to Inject Properties

```java
@Value("${spring.datasource.url}")
private String dbUrl;
```

or

```java
@ConfigurationProperties(prefix = "spring.datasource")
public class DataSourceSettings {
    private String url;
    private String username;
    private String password;
}
```

---

## ✅ Tip: Using Default Profile

If no profile is specified, Spring uses the **default** profile. You can override this with:

```properties
spring.profiles.default=dev
```

---

### ✅ Summary:

| Environment | File                          | Active Profile |
| ----------- | ----------------------------- | -------------- |
| Dev         | `application-dev.properties`  | `dev`          |
| Test        | `application-test.properties` | `test`         |
| Prod        | `application-prod.properties` | `prod`         |

Would you like an example Spring Boot project with 3 profiles (`dev`, `test`, `prod`) configured?
