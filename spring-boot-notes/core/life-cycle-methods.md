In Spring, **lifecycle methods** are used to perform custom initialization and destruction logic for beans. There are multiple ways to define these lifecycle methods:

---

### ✅ 1. **Using `@PostConstruct` and `@PreDestroy` Annotations (JSR-250)**

These annotations are the most common and modern approach.

```java
import jakarta.annotation.PostConstruct;
import jakarta.annotation.PreDestroy;
import org.springframework.stereotype.Component;

@Component
public class MyBean {

    @PostConstruct
    public void init() {
        System.out.println("Bean is going through init using @PostConstruct");
    }

    @PreDestroy
    public void destroy() {
        System.out.println("Bean will be destroyed using @PreDestroy");
    }
}
```

> **Note**: If you are using Spring Boot 3.x, use `jakarta.annotation.*`. For earlier versions, use `javax.annotation.*`.

---

### ✅ 2. **Implementing `InitializingBean` and `DisposableBean` Interfaces**

Spring provides these interfaces for custom init and destroy logic.

```java
import org.springframework.beans.factory.DisposableBean;
import org.springframework.beans.factory.InitializingBean;
import org.springframework.stereotype.Component;

@Component
public class MyBean implements InitializingBean, DisposableBean {

    @Override
    public void afterPropertiesSet() throws Exception {
        System.out.println("Bean is going through init using InitializingBean");
    }

    @Override
    public void destroy() throws Exception {
        System.out.println("Bean is being destroyed using DisposableBean");
    }
}
```

---

### ✅ 3. **Using `init-method` and `destroy-method` in `@Bean` Definition**

You can specify init and destroy methods directly in the `@Bean` configuration.

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class AppConfig {

    @Bean(initMethod = "customInit", destroyMethod = "customDestroy")
    public MyBean myBean() {
        return new MyBean();
    }
}

class MyBean {
    public void customInit() {
        System.out.println("Bean is going through custom init method");
    }

    public void customDestroy() {
        System.out.println("Bean will be destroyed using custom destroy method");
    }
}
```

---

### ✅ 4. **Using XML Configuration (Legacy)**

```xml
<bean id="myBean" class="com.example.MyBean"
      init-method="customInit" destroy-method="customDestroy"/>
```

And Java class:

```java
public class MyBean {
    public void customInit() {
        System.out.println("XML init method called");
    }

    public void customDestroy() {
        System.out.println("XML destroy method called");
    }
}
```

---

### 📝 Summary Table

| Method                               | Init Method | Destroy Method | Comments                                    |
| ------------------------------------ | ----------- | -------------- | ------------------------------------------- |
| `@PostConstruct`, `@PreDestroy`      | ✅           | ✅              | Most modern and recommended                 |
| `InitializingBean`, `DisposableBean` | ✅           | ✅              | Coupled with Spring, not POJO-friendly      |
| `@Bean(initMethod, destroyMethod)`   | ✅           | ✅              | Good for external classes                   |
| XML Configuration                    | ✅           | ✅              | Legacy; not used in modern Spring Boot apps |

---
