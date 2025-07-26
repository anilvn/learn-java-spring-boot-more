```java
// Main Application Class
@SpringBootApplication
public class BeanScopesApplication {
    public static void main(String[] args) {
        SpringApplication.run(BeanScopesApplication.class, args);
    }
}

// Service classes for different scopes
@Component
@Scope("singleton") // Default scope - same instance returned every time
public class SingletonService {
    private int counter = 0;
    
    public void increment() {
        counter++;
    }
    
    public int getCounter() {
        return counter;
    }
    
    public String getInstanceId() {
        return "SingletonService-" + this.hashCode();
    }
}

@Component
@Scope("prototype") // New instance created every time
public class PrototypeService {
    private int counter = 0;
    
    public void increment() {
        counter++;
    }
    
    public int getCounter() {
        return counter;
    }
    
    public String getInstanceId() {
        return "PrototypeService-" + this.hashCode();
    }
}

@Component
@Scope(value = WebApplicationContext.SCOPE_REQUEST, proxyMode = ScopedProxyMode.TARGET_CLASS)
// New instance per HTTP request
public class RequestScopedService {
    private int counter = 0;
    private final String requestId;
    
    public RequestScopedService() {
        this.requestId = "REQ-" + System.currentTimeMillis();
    }
    
    public void increment() {
        counter++;
    }
    
    public int getCounter() {
        return counter;
    }
    
    public String getRequestId() {
        return requestId;
    }
    
    public String getInstanceId() {
        return "RequestScopedService-" + this.hashCode();
    }
}

@Component
@Scope(value = WebApplicationContext.SCOPE_SESSION, proxyMode = ScopedProxyMode.TARGET_CLASS)
// New instance per HTTP session
public class SessionScopedService {
    private int counter = 0;
    private final String sessionId;
    
    public SessionScopedService() {
        this.sessionId = "SESSION-" + System.currentTimeMillis();
    }
    
    public void increment() {
        counter++;
    }
    
    public int getCounter() {
        return counter;
    }
    
    public String getSessionId() {
        return sessionId;
    }
    
    public String getInstanceId() {
        return "SessionScopedService-" + this.hashCode();
    }
}

// Controller to demonstrate the scopes
@RestController
@RequestMapping("/api/scopes")
public class ScopeController {
    
    @Autowired
    private SingletonService singletonService;
    
    @Autowired
    private ApplicationContext applicationContext;
    
    @Autowired
    private RequestScopedService requestScopedService;
    
    @Autowired
    private SessionScopedService sessionScopedService;
    
    @GetMapping("/singleton")
    public Map<String, Object> testSingleton() {
        singletonService.increment();
        
        // Get another instance to show it's the same
        SingletonService anotherInstance = applicationContext.getBean(SingletonService.class);
        
        Map<String, Object> result = new HashMap<>();
        result.put("counter", singletonService.getCounter());
        result.put("instanceId1", singletonService.getInstanceId());
        result.put("instanceId2", anotherInstance.getInstanceId());
        result.put("sameInstance", singletonService == anotherInstance);
        result.put("scope", "singleton");
        
        return result;
    }
    
    @GetMapping("/prototype")
    public Map<String, Object> testPrototype() {
        // Get two instances of prototype bean
        PrototypeService instance1 = applicationContext.getBean(PrototypeService.class);
        PrototypeService instance2 = applicationContext.getBean(PrototypeService.class);
        
        instance1.increment();
        instance1.increment();
        instance2.increment();
        
        Map<String, Object> result = new HashMap<>();
        result.put("instance1Counter", instance1.getCounter());
        result.put("instance2Counter", instance2.getCounter());
        result.put("instanceId1", instance1.getInstanceId());
        result.put("instanceId2", instance2.getInstanceId());
        result.put("sameInstance", instance1 == instance2);
        result.put("scope", "prototype");
        
        return result;
    }
    
    @GetMapping("/request")
    public Map<String, Object> testRequest() {
        requestScopedService.increment();
        
        Map<String, Object> result = new HashMap<>();
        result.put("counter", requestScopedService.getCounter());
        result.put("requestId", requestScopedService.getRequestId());
        result.put("instanceId", requestScopedService.getInstanceId());
        result.put("scope", "request");
        result.put("note", "Same instance within this request, new instance for each request");
        
        return result;
    }
    
    @GetMapping("/session")
    public Map<String, Object> testSession() {
        sessionScopedService.increment();
        
        Map<String, Object> result = new HashMap<>();
        result.put("counter", sessionScopedService.getCounter());
        result.put("sessionId", sessionScopedService.getSessionId());
        result.put("instanceId", sessionScopedService.getInstanceId());
        result.put("scope", "session");
        result.put("note", "Same instance within this session, new instance for each session");
        
        return result;
    }
    
    @GetMapping("/all")
    public Map<String, Object> testAll() {
        // Increment all services
        singletonService.increment();
        requestScopedService.increment();
        sessionScopedService.increment();
        
        // Get prototype instances
        PrototypeService proto1 = applicationContext.getBean(PrototypeService.class);
        PrototypeService proto2 = applicationContext.getBean(PrototypeService.class);
        proto1.increment();
        
        Map<String, Object> result = new HashMap<>();
        result.put("singleton", Map.of(
            "counter", singletonService.getCounter(),
            "instanceId", singletonService.getInstanceId()
        ));
        result.put("prototype", Map.of(
            "proto1Counter", proto1.getCounter(),
            "proto2Counter", proto2.getCounter(),
            "proto1InstanceId", proto1.getInstanceId(),
            "proto2InstanceId", proto2.getInstanceId()
        ));
        result.put("request", Map.of(
            "counter", requestScopedService.getCounter(),
            "requestId", requestScopedService.getRequestId(),
            "instanceId", requestScopedService.getInstanceId()
        ));
        result.put("session", Map.of(
            "counter", sessionScopedService.getCounter(),
            "sessionId", sessionScopedService.getSessionId(),
            "instanceId", sessionScopedService.getInstanceId()
        ));
        
        return result;
    }
}

// Configuration class (optional - shows alternative way to define beans)
@Configuration
public class BeanScopeConfig {
    
    @Bean
    @Scope("singleton")
    public String singletonBean() {
        return "Singleton Bean - " + System.currentTimeMillis();
    }
    
    @Bean
    @Scope("prototype")
    public String prototypeBean() {
        return "Prototype Bean - " + System.currentTimeMillis();
    }
    
    @Bean
    @Scope(value = WebApplicationContext.SCOPE_REQUEST, proxyMode = ScopedProxyMode.TARGET_CLASS)
    public String requestBean() {
        return "Request Bean - " + System.currentTimeMillis();
    }
    
    @Bean
    @Scope(value = WebApplicationContext.SCOPE_SESSION, proxyMode = ScopedProxyMode.TARGET_CLASS)
    public String sessionBean() {
        return "Session Bean - " + System.currentTimeMillis();
    }
}

// pom.xml dependencies needed:
/*
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter</artifactId>
    </dependency>
</dependencies>
*/
```