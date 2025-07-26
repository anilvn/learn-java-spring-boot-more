In Spring Framework, `ApplicationContext` is the central interface for providing configuration information to an application. It is a more advanced factory capable of managing the complete lifecycle of beans and providing features such as internationalization, event propagation, and resource loading.

### ✅ Common Implementations of `ApplicationContext`:

| Implementation Class                 | Description                                                   |
| ------------------------------------ | ------------------------------------------------------------- |
| `ClassPathXmlApplicationContext`     | Loads context definition from an XML file in the classpath.   |
| `FileSystemXmlApplicationContext`    | Loads context definition from an XML file in the file system. |
| `AnnotationConfigApplicationContext` | Loads context using annotated classes (Java Config).          |
| `WebApplicationContext`              | Specialized version for web applications (Spring MVC).        |

---

### 1. **ClassPathXmlApplicationContext**

```java
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class App {
    public static void main(String[] args) {
        ApplicationContext context = new ClassPathXmlApplicationContext("beans.xml");
        MyService service = context.getBean(MyService.class);
        service.doSomething();
    }
}
```

`beans.xml`:

```xml
<beans xmlns="http://www.springframework.org/schema/beans" ... >
    <bean id="myService" class="com.example.MyService"/>
</beans>
```

---

### 2. **FileSystemXmlApplicationContext**

```java
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.FileSystemXmlApplicationContext;

public class App {
    public static void main(String[] args) {
        ApplicationContext context = new FileSystemXmlApplicationContext("C:/config/beans.xml");
        MyService service = context.getBean(MyService.class);
        service.doSomething();
    }
}
```

---

### 3. **AnnotationConfigApplicationContext (Java-based Config)**

```java
import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.*;

@Configuration
class AppConfig {
    @Bean
    public MyService myService() {
        return new MyService();
    }
}

class MyService {
    public void doSomething() {
        System.out.println("Doing something...");
    }
}

public class App {
    public static void main(String[] args) {
        ApplicationContext context = new AnnotationConfigApplicationContext(AppConfig.class);
        MyService service = context.getBean(MyService.class);
        service.doSomething();
    }
}
```

---

### 4. **WebApplicationContext (Spring MVC)**

Used in web applications and configured in `web.xml` or via Java-based configuration with `DispatcherServlet`.

```xml
<!-- web.xml -->
<web-app ...>
    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>/WEB-INF/applicationContext.xml</param-value>
    </context-param>

    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>

    <servlet>
        <servlet-name>spring</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <load-on-startup>1</load-on-startup>
    </servlet>

    <servlet-mapping>
        <servlet-name>spring</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>
</web-app>
```

---

### Summary

| Implementation Class                 | Use Case                            |
| ------------------------------------ | ----------------------------------- |
| `ClassPathXmlApplicationContext`     | XML config in classpath             |
| `FileSystemXmlApplicationContext`    | XML config in file system path      |
| `AnnotationConfigApplicationContext` | Java-based @Configuration classes   |
| `WebApplicationContext`              | Web-specific context for Spring MVC |

