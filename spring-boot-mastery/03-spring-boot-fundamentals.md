# Chapter 3: Spring Boot Fundamentals
## พื้นฐานของ Spring Boot และ Auto-Configuration

### 🎯 Learning Objectives
หลังจากเรียนบทนี้แล้ว คุณจะสามารถ:
- เข้าใจหลักการทำงานของ Spring Boot Auto-Configuration
- รู้จักและใช้งาน Starter Dependencies อย่างมีประสิทธิภาพ
- จัดการ Configuration Properties และ Profiles
- เข้าใจ Application Context และ Bean Lifecycle
- ใช้งาน Spring Boot DevTools สำหรับการพัฒนา

---

## 🚀 Spring Boot Auto-Configuration Deep Dive

### **What is Auto-Configuration?**

Auto-Configuration เป็นหัวใจสำคัญของ Spring Boot ที่ช่วยลดการ configuration แบบ manual โดยการตรวจสอบ classpath และตั้งค่าอัตโนมัติ

#### **How Auto-Configuration Works**
```java
@SpringBootApplication
public class MyApplication {
    public static void main(String[] args) {
        SpringApplication.run(MyApplication.class, args);
    }
}

// @SpringBootApplication ประกอบด้วย:
@Configuration      // Configuration class
@EnableAutoConfiguration  // Enable auto-configuration
@ComponentScan     // Component scanning
```

#### **Auto-Configuration Process Flow**
```
1. Application Startup
   ↓
2. @EnableAutoConfiguration ถูกประมวลผล
   ↓
3. Spring Boot สแกน META-INF/spring.factories
   ↓
4. Load Auto-Configuration Classes
   ↓
5. Conditional Evaluation (@ConditionalOnClass, @ConditionalOnProperty, etc.)
   ↓
6. Bean Registration ตามเงื่อนไข
   ↓
7. Application Context Ready
```

### **Auto-Configuration Classes Examples**

#### **Web Auto-Configuration**
```java
@Configuration(proxyBeanMethods = false)
@ConditionalOnWebApplication(type = ConditionalOnWebApplication.Type.SERVLET)
@ConditionalOnClass({ Servlet.class, DispatcherServlet.class, WebMvcConfigurer.class })
@ConditionalOnMissingBean(WebMvcConfigurationSupport.class)
@AutoConfigureOrder(Ordered.HIGHEST_PRECEDENCE + 10)
@AutoConfigureAfter({ DispatcherServletAutoConfiguration.class, TaskExecutionAutoConfiguration.class,
		ValidationAutoConfiguration.class })
public class WebMvcAutoConfiguration {

    @Configuration(proxyBeanMethods = false)
    @Import(EnableWebMvcConfiguration.class)
    @EnableConfigurationProperties({ WebMvcProperties.class, ResourceProperties.class })
    @Order(0)
    public static class WebMvcAutoConfigurationAdapter implements WebMvcConfigurer {

        @Bean
        @ConditionalOnMissingBean
        public InternalResourceViewResolver defaultViewResolver() {
            InternalResourceViewResolver resolver = new InternalResourceViewResolver();
            resolver.setPrefix("/WEB-INF/views/");
            resolver.setSuffix(".jsp");
            return resolver;
        }
    }
}
```

#### **DataSource Auto-Configuration**
```java
@Configuration(proxyBeanMethods = false)
@ConditionalOnClass({ DataSource.class, EmbeddedDatabaseType.class })
@ConditionalOnMissingBean(type = "javax.sql.DataSource")
@EnableConfigurationProperties(DataSourceProperties.class)
@Import({ DataSourcePoolMetadataProvidersConfiguration.class,
          DataSourceInitializationConfiguration.class })
public class DataSourceAutoConfiguration {

    @Configuration(proxyBeanMethods = false)
    @Conditional(EmbeddedDatabaseCondition.class)
    @ConditionalOnMissingBean({ DataSource.class, XADataSource.class })
    @Import(EmbeddedDataSourceConfiguration.class)
    protected static class EmbeddedDatabaseConfiguration {
        // H2, HSQL, Derby auto-configuration
    }

    @Configuration(proxyBeanMethods = false)
    @Conditional(PooledDataSourceCondition.class)
    @ConditionalOnMissingBean({ DataSource.class, XADataSource.class })
    @Import({ HikariConfiguration.class, TomcatConfiguration.class,
              DbcpConfiguration.class, GenericConfiguration.class })
    protected static class PooledDataSourceConfiguration {
        // Connection pool auto-configuration
    }
}
```

### **Conditional Annotations**

#### **Common Conditional Annotations**
```java
// Class-based conditions
@ConditionalOnClass(DataSource.class)           // If class exists in classpath
@ConditionalOnMissingClass("com.example.SomeClass")  // If class doesn't exist

// Bean-based conditions  
@ConditionalOnBean(DataSource.class)            // If bean exists
@ConditionalOnMissingBean(DataSource.class)     // If bean doesn't exist

// Property-based conditions
@ConditionalOnProperty(                         // If property matches
    name = "spring.datasource.enabled",
    havingValue = "true",
    matchIfMissing = true
)

// Resource-based conditions
@ConditionalOnResource(resources = "classpath:db.properties")

// Expression-based conditions
@ConditionalOnExpression("${server.port:8080} > 8000")

// Web application type
@ConditionalOnWebApplication(type = ConditionalOnWebApplication.Type.SERVLET)
@ConditionalOnNotWebApplication
```

#### **Custom Conditional Configuration**
```java
@Configuration
public class DatabaseConfiguration {

    @Bean
    @ConditionalOnProperty(name = "app.database.type", havingValue = "mysql")
    @ConditionalOnClass(name = "com.mysql.cj.jdbc.Driver")
    public DataSource mysqlDataSource() {
        HikariConfig config = new HikariConfig();
        config.setDriverClassName("com.mysql.cj.jdbc.Driver");
        config.setJdbcUrl("jdbc:mysql://localhost:3306/mydb");
        config.setUsername("user");
        config.setPassword("password");
        return new HikariDataSource(config);
    }

    @Bean
    @ConditionalOnProperty(name = "app.database.type", havingValue = "h2", matchIfMissing = true)
    @ConditionalOnClass(name = "org.h2.Driver")
    public DataSource h2DataSource() {
        return new EmbeddedDatabaseBuilder()
                .setType(EmbeddedDatabaseType.H2)
                .addScript("schema.sql")
                .addScript("data.sql")
                .build();
    }
}
```

---

## 📦 Starter Dependencies Deep Dive

### **What are Starter Dependencies?**

Starter Dependencies เป็น dependency descriptors ที่รวม libraries ที่เกี่ยวข้องไว้ด้วยกัน เพื่อให้ใช้งานง่ายขึ้น

#### **Starter Dependency Structure**
```xml
<!-- spring-boot-starter-web includes: -->
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-tomcat</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-webmvc</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-web</artifactId>
    </dependency>
    <!-- และอื่นๆ -->
</dependencies>
```

### **Common Starter Dependencies**

#### **Core Starters**
```xml
<!-- Basic Spring Boot functionality -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter</artifactId>
</dependency>

<!-- Web applications with Spring MVC -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>

<!-- Reactive web applications with WebFlux -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-webflux</artifactId>
</dependency>

<!-- Testing framework -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>
```

#### **Data Access Starters**
```xml
<!-- JPA with Hibernate -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>

<!-- MongoDB -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-mongodb</artifactId>
</dependency>

<!-- Redis -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>

<!-- JDBC -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>
```

#### **Security and Authentication**
```xml
<!-- Spring Security -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>

<!-- OAuth2 Client -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-oauth2-client</artifactId>
</dependency>

<!-- OAuth2 Resource Server -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-oauth2-resource-server</artifactId>
</dependency>
```

#### **Cloud and Microservices**
```xml
<!-- Spring Cloud -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter</artifactId>
</dependency>

<!-- Service Discovery (Eureka) -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>

<!-- API Gateway -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-gateway</artifactId>
</dependency>

<!-- Circuit Breaker -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-circuitbreaker-resilience4j</artifactId>
</dependency>
```

### **Creating Custom Starter**

#### **Custom Starter Structure**
```
my-custom-starter/
├── pom.xml
└── src/
    └── main/
        ├── java/
        │   └── com/example/starter/
        │       ├── MyAutoConfiguration.java
        │       ├── MyProperties.java
        │       └── MyService.java
        └── resources/
            └── META-INF/
                └── spring.factories
```

#### **Custom Starter Implementation**
```java
// MyProperties.java
@ConfigurationProperties(prefix = "my.service")
public class MyProperties {
    private boolean enabled = true;
    private String apiKey;
    private int timeout = 5000;
    
    // Getters and setters
    public boolean isEnabled() { return enabled; }
    public void setEnabled(boolean enabled) { this.enabled = enabled; }
    
    public String getApiKey() { return apiKey; }
    public void setApiKey(String apiKey) { this.apiKey = apiKey; }
    
    public int getTimeout() { return timeout; }
    public void setTimeout(int timeout) { this.timeout = timeout; }
}

// MyService.java
public class MyService {
    private final MyProperties properties;
    
    public MyService(MyProperties properties) {
        this.properties = properties;
    }
    
    public String performOperation(String input) {
        if (!properties.isEnabled()) {
            throw new IllegalStateException("Service is disabled");
        }
        
        // Perform operation with timeout and API key
        return "Processed: " + input + " with key: " + properties.getApiKey();
    }
}

// MyAutoConfiguration.java
@Configuration
@EnableConfigurationProperties(MyProperties.class)
@ConditionalOnProperty(name = "my.service.enabled", havingValue = "true", matchIfMissing = true)
public class MyAutoConfiguration {
    
    @Bean
    @ConditionalOnMissingBean
    public MyService myService(MyProperties properties) {
        return new MyService(properties);
    }
}
```

#### **spring.factories Configuration**
```properties
# META-INF/spring.factories
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
com.example.starter.MyAutoConfiguration
```

#### **Custom Starter POM**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0">
    <modelVersion>4.0.0</modelVersion>
    
    <groupId>com.example</groupId>
    <artifactId>my-service-spring-boot-starter</artifactId>
    <version>1.0.0</version>
    <packaging>jar</packaging>
    
    <name>My Service Spring Boot Starter</name>
    <description>Custom Spring Boot Starter for My Service</description>
    
    <properties>
        <java.version>17</java.version>
        <spring-boot.version>3.2.0</spring-boot.version>
    </properties>
    
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-autoconfigure</artifactId>
            <version>${spring-boot.version}</version>
        </dependency>
        
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-configuration-processor</artifactId>
            <version>${spring-boot.version}</version>
            <optional>true</optional>
        </dependency>
    </dependencies>
</project>
```

---

## ⚙️ Configuration Properties and Profiles

### **Configuration Properties**

#### **application.properties vs application.yml**
```properties
# application.properties
server.port=8080
server.servlet.context-path=/api

spring.datasource.url=jdbc:mysql://localhost:3306/mydb
spring.datasource.username=user
spring.datasource.password=password
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver

spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL8Dialect

logging.level.com.example=DEBUG
logging.level.org.springframework.web=INFO
```

```yaml
# application.yml (Recommended)
server:
  port: 8080
  servlet:
    context-path: /api

spring:
  datasource:
    url: jdbc:mysql://localhost:3306/mydb
    username: user
    password: password
    driver-class-name: com.mysql.cj.jdbc.Driver
  
  jpa:
    hibernate:
      ddl-auto: update
    show-sql: true
    properties:
      hibernate:
        dialect: org.hibernate.dialect.MySQL8Dialect

logging:
  level:
    com.example: DEBUG
    org.springframework.web: INFO
```

### **Custom Configuration Properties**

#### **@ConfigurationProperties Approach**
```java
@ConfigurationProperties(prefix = "app")
@Component
public class AppProperties {
    
    private String name;
    private String version;
    private Security security = new Security();
    private Database database = new Database();
    
    // Getters and setters
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
    
    public String getVersion() { return version; }
    public void setVersion(String version) { this.version = version; }
    
    public Security getSecurity() { return security; }
    public void setSecurity(Security security) { this.security = security; }
    
    public Database getDatabase() { return database; }
    public void setDatabase(Database database) { this.database = database; }
    
    public static class Security {
        private boolean enabled = true;
        private String secretKey;
        private int tokenExpirationHours = 24;
        
        // Getters and setters
        public boolean isEnabled() { return enabled; }
        public void setEnabled(boolean enabled) { this.enabled = enabled; }
        
        public String getSecretKey() { return secretKey; }
        public void setSecretKey(String secretKey) { this.secretKey = secretKey; }
        
        public int getTokenExpirationHours() { return tokenExpirationHours; }
        public void setTokenExpirationHours(int tokenExpirationHours) { 
            this.tokenExpirationHours = tokenExpirationHours; 
        }
    }
    
    public static class Database {
        private int maxConnections = 20;
        private int connectionTimeout = 30000;
        private boolean showSql = false;
        
        // Getters and setters
        public int getMaxConnections() { return maxConnections; }
        public void setMaxConnections(int maxConnections) { this.maxConnections = maxConnections; }
        
        public int getConnectionTimeout() { return connectionTimeout; }
        public void setConnectionTimeout(int connectionTimeout) { this.connectionTimeout = connectionTimeout; }
        
        public boolean isShowSql() { return showSql; }
        public void setShowSql(boolean showSql) { this.showSql = showSql; }
    }
}
```

#### **Configuration YAML for Custom Properties**
```yaml
app:
  name: My Spring Boot Application
  version: 1.0.0
  security:
    enabled: true
    secret-key: ${APP_SECRET_KEY:default-secret-key}
    token-expiration-hours: 24
  database:
    max-connections: 20
    connection-timeout: 30000
    show-sql: false
```

#### **Using Configuration Properties**
```java
@Service
public class UserService {
    
    private final AppProperties appProperties;
    
    public UserService(AppProperties appProperties) {
        this.appProperties = appProperties;
    }
    
    public String generateToken(User user) {
        if (!appProperties.getSecurity().isEnabled()) {
            throw new SecurityException("Security is disabled");
        }
        
        String secretKey = appProperties.getSecurity().getSecretKey();
        int expirationHours = appProperties.getSecurity().getTokenExpirationHours();
        
        // Generate JWT token with secret key and expiration
        return JwtUtils.generateToken(user, secretKey, expirationHours);
    }
}
```

### **Profiles Management**

#### **Profile-Specific Configuration Files**
```
src/main/resources/
├── application.yml              # Default configuration
├── application-dev.yml          # Development environment
├── application-staging.yml      # Staging environment
├── application-prod.yml         # Production environment
└── application-test.yml         # Test environment
```

#### **Profile Configuration Examples**
```yaml
# application.yml (Default)
spring:
  profiles:
    active: dev

app:
  name: My Application
  debug: false

---
# application-dev.yml
spring:
  datasource:
    url: jdbc:h2:mem:devdb
    driver-class-name: org.h2.Driver
    username: sa
    password: password
  
  h2:
    console:
      enabled: true
  
  jpa:
    hibernate:
      ddl-auto: create-drop
    show-sql: true

app:
  debug: true

logging:
  level:
    com.example: DEBUG
    org.springframework: INFO

---
# application-prod.yml
spring:
  datasource:
    url: jdbc:postgresql://${DB_HOST:localhost}:${DB_PORT:5432}/${DB_NAME:proddb}
    username: ${DB_USERNAME}
    password: ${DB_PASSWORD}
    driver-class-name: org.postgresql.Driver
  
  jpa:
    hibernate:
      ddl-auto: validate
    show-sql: false

app:
  debug: false

logging:
  level:
    com.example: INFO
    org.springframework: WARN
  file:
    name: /var/log/myapp.log

management:
  endpoints:
    web:
      exposure:
        include: health,info,metrics
```

#### **Profile-Specific Beans**
```java
@Configuration
public class DatabaseConfiguration {
    
    @Bean
    @Profile("dev")
    public DataSource devDataSource() {
        return new EmbeddedDatabaseBuilder()
                .setType(EmbeddedDatabaseType.H2)
                .addScript("schema.sql")
                .addScript("test-data.sql")
                .build();
    }
    
    @Bean
    @Profile("prod")
    public DataSource prodDataSource() {
        HikariConfig config = new HikariConfig();
        config.setDriverClassName("org.postgresql.Driver");
        config.setJdbcUrl(System.getenv("DATABASE_URL"));
        config.setUsername(System.getenv("DB_USERNAME"));
        config.setPassword(System.getenv("DB_PASSWORD"));
        config.setMaximumPoolSize(20);
        return new HikariDataSource(config);
    }
    
    @Bean
    @Profile("test")
    public DataSource testDataSource() {
        return new EmbeddedDatabaseBuilder()
                .setType(EmbeddedDatabaseType.H2)
                .addScript("schema.sql")
                .build();
    }
}
```

#### **Activating Profiles**
```bash
# Via application.properties
spring.profiles.active=dev,debug

# Via command line
java -jar myapp.jar --spring.profiles.active=prod

# Via environment variable
export SPRING_PROFILES_ACTIVE=prod,monitoring

# Via JVM system property
java -Dspring.profiles.active=prod -jar myapp.jar

# Programmatically
@SpringBootApplication
public class MyApplication {
    public static void main(String[] args) {
        SpringApplication app = new SpringApplication(MyApplication.class);
        app.setAdditionalProfiles("dev");
        app.run(args);
    }
}
```

---

## 🔄 Application Context and Bean Lifecycle

### **Spring Application Context**

#### **Application Context Hierarchy**
```java
@SpringBootApplication
public class MyApplication {
    
    public static void main(String[] args) {
        ConfigurableApplicationContext context = SpringApplication.run(MyApplication.class, args);
        
        // Get bean from context
        UserService userService = context.getBean(UserService.class);
        
        // Get all beans of type
        Map<String, UserService> userServices = context.getBeansOfType(UserService.class);
        
        // Check if bean exists
        boolean hasUserService = context.containsBean("userService");
        
        // Get bean names
        String[] beanNames = context.getBeanDefinitionNames();
        
        // Close context gracefully
        context.close();
    }
}
```

#### **Bean Scopes**
```java
@Component
@Scope("singleton")  // Default scope
public class SingletonService {
    // One instance per application context
}

@Component
@Scope("prototype")  // New instance every time
public class PrototypeService {
    // New instance for each request
}

@Component
@Scope("request")    // Web applications only
public class RequestScopedService {
    // One instance per HTTP request
}

@Component
@Scope("session")    // Web applications only
public class SessionScopedService {
    // One instance per HTTP session
}

@Component
@Scope("application") // Web applications only
public class ApplicationScopedService {
    // One instance per ServletContext
}
```

### **Bean Lifecycle Management**

#### **Bean Lifecycle Hooks**
```java
@Component
public class LifecycleAwareService implements InitializingBean, DisposableBean {
    
    private static final Logger log = LoggerFactory.getLogger(LifecycleAwareService.class);
    
    // Constructor
    public LifecycleAwareService() {
        log.info("1. Constructor called");
    }
    
    // Dependency injection after construction
    @Autowired
    public void setDependency(SomeDependency dependency) {
        log.info("2. Dependencies injected");
    }
    
    // Post-construct initialization
    @PostConstruct
    public void init() {
        log.info("3. @PostConstruct called");
        // Initialize resources, validate configuration, etc.
    }
    
    // InitializingBean interface method
    @Override
    public void afterPropertiesSet() throws Exception {
        log.info("4. afterPropertiesSet() called");
        // Additional initialization after all properties are set
    }
    
    // Custom init method (configured in @Bean annotation)
    public void customInit() {
        log.info("5. Custom init method called");
    }
    
    // Business methods
    public void doSomething() {
        log.info("Performing business operation");
    }
    
    // Pre-destroy cleanup
    @PreDestroy
    public void cleanup() {
        log.info("6. @PreDestroy called");
        // Clean up resources, close connections, etc.
    }
    
    // DisposableBean interface method
    @Override
    public void destroy() throws Exception {
        log.info("7. destroy() called");
        // Additional cleanup
    }
}
```

#### **Configuration-Based Lifecycle Management**
```java
@Configuration
public class ServiceConfiguration {
    
    @Bean(initMethod = "customInit", destroyMethod = "customDestroy")
    public ExternalService externalService() {
        ExternalService service = new ExternalService();
        service.setApiKey("api-key");
        service.setTimeout(5000);
        return service;
    }
}

public class ExternalService {
    private String apiKey;
    private int timeout;
    
    public void customInit() {
        // Initialize connection
        log.info("Initializing external service connection");
    }
    
    public void customDestroy() {
        // Close connection
        log.info("Closing external service connection");
    }
    
    // Getters and setters
    public void setApiKey(String apiKey) { this.apiKey = apiKey; }
    public void setTimeout(int timeout) { this.timeout = timeout; }
}
```

### **Application Events**

#### **Built-in Application Events**
```java
@Component
public class ApplicationEventListener {
    
    private static final Logger log = LoggerFactory.getLogger(ApplicationEventListener.class);
    
    @EventListener
    public void handleApplicationStarted(ApplicationStartedEvent event) {
        log.info("Application started in {} ms", 
                event.getTimeTaken().toMillis());
    }
    
    @EventListener
    public void handleApplicationReady(ApplicationReadyEvent event) {
        log.info("Application ready to serve requests");
        // Perform post-startup tasks
    }
    
    @EventListener
    public void handleContextRefreshed(ContextRefreshedEvent event) {
        log.info("Application context refreshed");
    }
    
    @EventListener
    public void handleContextClosed(ContextClosedEvent event) {
        log.info("Application context closing");
        // Perform cleanup before shutdown
    }
}
```

#### **Custom Application Events**
```java
// Custom event
public class UserRegistrationEvent extends ApplicationEvent {
    private final User user;
    private final String registrationSource;
    
    public UserRegistrationEvent(Object source, User user, String registrationSource) {
        super(source);
        this.user = user;
        this.registrationSource = registrationSource;
    }
    
    public User getUser() { return user; }
    public String getRegistrationSource() { return registrationSource; }
}

// Event publisher
@Service
public class UserService {
    
    private final ApplicationEventPublisher eventPublisher;
    
    public UserService(ApplicationEventPublisher eventPublisher) {
        this.eventPublisher = eventPublisher;
    }
    
    public User registerUser(UserRegistrationRequest request) {
        User user = new User(request.getUsername(), request.getEmail());
        
        // Save user to database
        User savedUser = userRepository.save(user);
        
        // Publish event
        eventPublisher.publishEvent(
            new UserRegistrationEvent(this, savedUser, "web")
        );
        
        return savedUser;
    }
}

// Event listeners
@Component
public class UserRegistrationEventHandler {
    
    @EventListener
    @Async
    public void handleUserRegistration(UserRegistrationEvent event) {
        User user = event.getUser();
        
        // Send welcome email
        emailService.sendWelcomeEmail(user);
        
        // Update analytics
        analyticsService.trackUserRegistration(user, event.getRegistrationSource());
        
        // Initialize user preferences
        userPreferencesService.createDefaultPreferences(user);
    }
}
```

---

## 🛠️ Spring Boot DevTools

### **DevTools Features**

#### **DevTools Installation**
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
    <scope>runtime</scope>
    <optional>true</optional>
</dependency>
```

#### **Automatic Restart**
```yaml
# application-dev.yml
spring:
  devtools:
    restart:
      enabled: true
      additional-paths: src/main/resources
      exclude: static/**,public/**
      poll-interval: 1000
      quiet-period: 400
```

#### **Live Reload**
```yaml
spring:
  devtools:
    livereload:
      enabled: true
      port: 35729
```

#### **Remote Development**
```yaml
# Enable remote development
spring:
  devtools:
    remote:
      secret: mysecret
      context-path: /.~~spring-boot!~
      restart:
        enabled: true
```

### **DevTools Configuration**

#### **Global DevTools Properties**
```properties
# ~/.spring-boot-devtools.properties (Global configuration)
spring.devtools.restart.trigger-file=.reloadtrigger
spring.devtools.restart.additional-exclude=logs/**
spring.devtools.livereload.enabled=true
```

#### **Excluding Resources from Restart**
```java
@Configuration
public class DevToolsConfiguration {
    
    @Bean
    @Profile("dev")
    public FileSystemWatcher fileSystemWatcher() {
        FileSystemWatcher watcher = new FileSystemWatcher();
        watcher.addSourceDirectory(new File("src/main/java"));
        watcher.addSourceDirectory(new File("src/main/resources"));
        return watcher;
    }
}
```

---

## 🏗️ Understanding @SpringBootApplication

### **@SpringBootApplication Breakdown**

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = { 
    @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
    @Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) 
})
public @interface SpringBootApplication {
    
    @AliasFor(annotation = EnableAutoConfiguration.class)
    Class<?>[] exclude() default {};
    
    @AliasFor(annotation = EnableAutoConfiguration.class)
    String[] excludeName() default {};
    
    @AliasFor(annotation = ComponentScan.class, attribute = "basePackages")
    String[] scanBasePackages() default {};
    
    @AliasFor(annotation = ComponentScan.class, attribute = "basePackageClasses")
    Class<?>[] scanBasePackageClasses() default {};
    
    boolean proxyBeanMethods() default true;
}
```

#### **Customizing @SpringBootApplication**
```java
@SpringBootApplication(
    exclude = {DataSourceAutoConfiguration.class, SecurityAutoConfiguration.class},
    scanBasePackages = {"com.example.myapp", "com.example.shared"}
)
public class MyApplication {
    
    public static void main(String[] args) {
        SpringApplication app = new SpringApplication(MyApplication.class);
        
        // Customize application
        app.setDefaultProperties(Collections.singletonMap(
            "server.port", "8081"
        ));
        
        // Add additional profiles
        app.setAdditionalProfiles("custom");
        
        // Disable banner
        app.setBannerMode(Banner.Mode.OFF);
        
        // Set web application type
        app.setWebApplicationType(WebApplicationType.SERVLET);
        
        app.run(args);
    }
}
```

#### **Separating Configuration Classes**
```java
// Main application class - minimal
@SpringBootApplication
public class MyApplication {
    public static void main(String[] args) {
        SpringApplication.run(MyApplication.class, args);
    }
}

// Database configuration
@Configuration
@EnableJpaRepositories(basePackages = "com.example.repository")
@EntityScan(basePackages = "com.example.domain")
public class DatabaseConfiguration {
    
    @Bean
    @Primary
    public DataSource primaryDataSource() {
        // Primary database configuration
    }
    
    @Bean
    @Qualifier("secondary")
    public DataSource secondaryDataSource() {
        // Secondary database configuration
    }
}

// Web configuration
@Configuration
@EnableWebMvc
public class WebConfiguration implements WebMvcConfigurer {
    
    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/api/**")
                .allowedOrigins("http://localhost:3000")
                .allowedMethods("GET", "POST", "PUT", "DELETE")
                .allowCredentials(true);
    }
    
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new LoggingInterceptor());
    }
}
```

---

## 📊 Application Properties Reference

### **Core Spring Boot Properties**

#### **Server Configuration**
```yaml
server:
  port: 8080                          # Server port
  servlet:
    context-path: /api                # Context path
    session:
      timeout: 30m                    # Session timeout
  compression:
    enabled: true                     # Enable response compression
    mime-types: text/html,text/xml,text/plain,text/css,application/json
  ssl:
    enabled: false                    # Enable SSL
    key-store: classpath:keystore.p12
    key-store-password: password
    key-store-type: PKCS12
```

#### **Actuator Configuration**
```yaml
management:
  endpoints:
    web:
      exposure:
        include: health,info,metrics,env,configprops,beans
  endpoint:
    health:
      show-details: always
    info:
      enabled: true
  info:
    env:
      enabled: true
    git:
      mode: full
```

#### **Logging Configuration**
```yaml
logging:
  level:
    org.springframework: INFO
    com.example: DEBUG
    org.hibernate.SQL: DEBUG
    org.hibernate.type.descriptor.sql.BasicBinder: TRACE
  pattern:
    console: "%d{yyyy-MM-dd HH:mm:ss} - %msg%n"
    file: "%d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level %logger{36} - %msg%n"
  file:
    name: application.log
    max-size: 10MB
    max-history: 30
```

#### **Database Configuration**
```yaml
spring:
  datasource:
    url: jdbc:postgresql://localhost:5432/mydb
    username: ${DB_USERNAME:user}
    password: ${DB_PASSWORD:password}
    driver-class-name: org.postgresql.Driver
    hikari:
      connection-timeout: 20000
      maximum-pool-size: 20
      minimum-idle: 5
      idle-timeout: 300000
      max-lifetime: 1200000
  
  jpa:
    hibernate:
      ddl-auto: validate
      naming:
        physical-strategy: org.hibernate.boot.model.naming.PhysicalNamingStrategyStandardImpl
    show-sql: false
    properties:
      hibernate:
        dialect: org.hibernate.dialect.PostgreSQLDialect
        format_sql: true
        use_sql_comments: true
        jdbc:
          batch_size: 25
          order_inserts: true
          order_updates: true
```

---

## 🧪 Practical Examples

### **Complete Configuration Example**

#### **Main Application Class**
```java
@SpringBootApplication
public class ECommerceApplication {
    
    public static void main(String[] args) {
        SpringApplication.run(ECommerceApplication.class, args);
    }
    
    @Bean
    public CommandLineRunner initData(UserRepository userRepository) {
        return args -> {
            if (userRepository.count() == 0) {
                User admin = new User("admin", "admin@example.com", Role.ADMIN);
                userRepository.save(admin);
                log.info("Admin user created");
            }
        };
    }
}
```

#### **Custom Properties Configuration**
```java
@ConfigurationProperties(prefix = "ecommerce")
@Component
public class ECommerceProperties {
    
    private String appName = "E-Commerce Platform";
    private String version = "1.0.0";
    private Payment payment = new Payment();
    private Email email = new Email();
    private Cache cache = new Cache();
    
    // Getters and setters
    
    public static class Payment {
        private String provider = "stripe";
        private String apiKey;
        private String secretKey;
        private boolean sandbox = true;
        
        // Getters and setters
    }
    
    public static class Email {
        private String provider = "smtp";
        private String host = "smtp.gmail.com";
        private int port = 587;
        private String username;
        private String password;
        private boolean enableTls = true;
        
        // Getters and setters
    }
    
    public static class Cache {
        private String provider = "redis";
        private String host = "localhost";
        private int port = 6379;
        private int database = 0;
        private int ttlHours = 24;
        
        // Getters and setters
    }
}
```

#### **Application Configuration**
```yaml
# application.yml
spring:
  profiles:
    active: dev
  application:
    name: ecommerce-platform

ecommerce:
  app-name: E-Commerce Platform
  version: 1.0.0
  payment:
    provider: stripe
    api-key: ${STRIPE_API_KEY:pk_test_123}
    secret-key: ${STRIPE_SECRET_KEY:sk_test_123}
    sandbox: true
  email:
    provider: smtp
    host: ${EMAIL_HOST:smtp.gmail.com}
    port: ${EMAIL_PORT:587}
    username: ${EMAIL_USERNAME}
    password: ${EMAIL_PASSWORD}
    enable-tls: true
  cache:
    provider: redis
    host: ${REDIS_HOST:localhost}
    port: ${REDIS_PORT:6379}
    database: 0
    ttl-hours: 24

---
# Development profile
spring:
  config:
    activate:
      on-profile: dev
  datasource:
    url: jdbc:h2:mem:devdb
    driver-class-name: org.h2.Driver
  h2:
    console:
      enabled: true
  jpa:
    hibernate:
      ddl-auto: create-drop
    show-sql: true

logging:
  level:
    com.example: DEBUG

---
# Production profile
spring:
  config:
    activate:
      on-profile: prod
  datasource:
    url: jdbc:postgresql://${DB_HOST}:${DB_PORT}/${DB_NAME}
    username: ${DB_USERNAME}
    password: ${DB_PASSWORD}
    driver-class-name: org.postgresql.Driver
  jpa:
    hibernate:
      ddl-auto: validate
    show-sql: false

logging:
  level:
    com.example: INFO
    org.springframework: WARN

ecommerce:
  payment:
    sandbox: false
```

---

## 📚 Summary and Best Practices

### **Key Takeaways**

#### **Auto-Configuration**
- ✅ เข้าใจว่า Spring Boot ทำ auto-configuration อย่างไร
- ✅ รู้จักใช้ Conditional annotations
- ✅ สามารถสร้าง custom auto-configuration ได้

#### **Starter Dependencies**
- ✅ เลือกใช้ starter ที่เหมาะสมกับงาน
- ✅ เข้าใจ dependency management
- ✅ สร้าง custom starter ได้

#### **Configuration Management**
- ✅ ใช้ YAML แทน Properties files
- ✅ จัดการ profiles อย่างมีประสิทธิภาพ
- ✅ ใช้ @ConfigurationProperties สำหรับ custom properties

#### **Application Lifecycle**
- ✅ เข้าใจ bean lifecycle และ scopes
- ✅ ใช้ application events อย่างเหมาะสม
- ✅ จัดการ initialization และ cleanup ถูกต้อง

### **Best Practices**

#### **Configuration Best Practices**
```yaml
# ✅ Good - Environment-specific externalized config
spring:
  datasource:
    url: ${DATABASE_URL}
    username: ${DB_USERNAME}
    password: ${DB_PASSWORD}

# ❌ Avoid - Hard-coded values
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/mydb
    username: root
    password: password123
```

#### **Properties Organization**
```java
// ✅ Good - Organized configuration properties
@ConfigurationProperties(prefix = "app")
public class AppProperties {
    private Database database = new Database();
    private Security security = new Security();
    private Email email = new Email();
    
    // Nested classes for organization
}

// ❌ Avoid - Flat properties
@ConfigurationProperties(prefix = "app")
public class AppProperties {
    private String databaseUrl;
    private String databaseUsername;
    private String securitySecretKey;
    private String emailHost;
    // Too many flat properties
}
```

#### **Profile Management**
```yaml
# ✅ Good - Clear profile separation
---
spring:
  config:
    activate:
      on-profile: dev
# Development-specific config

---
spring:
  config:
    activate:
      on-profile: prod
# Production-specific config
```

### **Common Pitfalls to Avoid**

#### **Auto-Configuration Issues**
```java
// ❌ Avoid - Disabling too much auto-configuration
@SpringBootApplication(exclude = {
    DataSourceAutoConfiguration.class,
    SecurityAutoConfiguration.class,
    WebMvcAutoConfiguration.class
    // Don't exclude unless you really need to
})

// ✅ Good - Selective exclusion when needed
@SpringBootApplication(exclude = DataSourceAutoConfiguration.class)
// Only exclude what you're replacing
```

#### **Bean Definition Issues**
```java
// ❌ Avoid - Conflicting bean definitions
@Bean
public DataSource dataSource1() { ... }

@Bean
public DataSource dataSource2() { ... }
// Will cause conflicts

// ✅ Good - Use @Primary or @Qualifier
@Bean
@Primary
public DataSource primaryDataSource() { ... }

@Bean
@Qualifier("secondary")
public DataSource secondaryDataSource() { ... }
```

---

## 🏋️ Practice Exercises

### **Exercise 1: Custom Configuration Properties**
1. Create a custom `@ConfigurationProperties` class for a blog application
2. Include nested properties for database, email, and file storage
3. Use different values for dev and prod profiles
4. Validate properties using Bean Validation annotations

### **Exercise 2: Custom Auto-Configuration**
1. Create a custom starter for a notification service
2. Implement auto-configuration with conditional logic
3. Support multiple notification providers (email, SMS, push)
4. Include configuration properties and default values

### **Exercise 3: Application Events**
1. Create custom events for user actions (login, logout, profile update)
2. Implement event listeners for audit logging
3. Use async event processing
4. Test event publishing and handling

### **Exercise 4: Profile Management**
1. Set up profiles for local, dev, staging, and prod environments
2. Configure different databases for each profile
3. Use environment variables for sensitive configuration
4. Test profile activation and property resolution

---

**🎯 You now understand Spring Boot fundamentals! Ready for the next level?**

**[Next Chapter: Spring Core Concepts →](./04-spring-core-concepts.md)**

---

*Duration: ~3 hours | Difficulty: Beginner to Intermediate | Prerequisites: Chapter 2 completed*
