# Chapter 1: Java & Spring Introduction
## บทนำสู่ Java และ Spring Framework

### 🎯 Learning Objectives
หลังจากเรียนบทนี้แล้ว คุณจะสามารถ:
- เข้าใจประวัติและวิวัฒนาการของ Java และ Spring Framework
- เข้าใจ Java Virtual Machine (JVM) และ ecosystem
- รู้จักกับ Spring Framework และ Spring Boot
- เข้าใจความแตกต่างระหว่าง Spring Framework กับ Spring Boot
- เตรียมความพร้อมสำหรับการเรียนรู้ Spring Boot

---

## 🌟 Java Platform Overview

### **Java Platform Evolution**

Java เป็นภาษาโปรแกรมมิ่งที่พัฒนาโดย Sun Microsystems (ปัจจุบันเป็นของ Oracle) ซึ่งมีหลักการสำคัญคือ **"Write Once, Run Anywhere" (WORA)**

#### **Java Version Timeline**
```
Java 1.0 (1996) → Java 1.1 (1997) → Java 1.2 (1998) → ... →
Java 8 (2014) → Java 11 (2018) → Java 17 (2021) → Java 21 (2023)
```

#### **Key Java Features**
- ✅ **Platform Independence** - JVM abstraction layer
- ✅ **Object-Oriented** - Encapsulation, Inheritance, Polymorphism
- ✅ **Memory Management** - Automatic garbage collection
- ✅ **Multi-threading** - Concurrent programming support
- ✅ **Rich Ecosystem** - Extensive library and framework support

### **Java Virtual Machine (JVM) Architecture**

```
┌─────────────────────────────────────────────────────────┐
│                    Java Application                     │
├─────────────────────────────────────────────────────────┤
│                Java Runtime Environment                 │
│  ┌─────────────────┐  ┌─────────────────────────────┐   │
│  │   Java API      │  │      JVM Components         │   │
│  │   - Collections │  │  ┌─────────────────────────┐ │   │
│  │   - I/O         │  │  │    Class Loader         │ │   │
│  │   - Networking  │  │  ├─────────────────────────┤ │   │
│  │   - Security    │  │  │    Execution Engine     │ │   │
│  │   - Concurrency │  │  │    - Interpreter        │ │   │
│  │                 │  │  │    - JIT Compiler       │ │   │
│  └─────────────────┘  │  ├─────────────────────────┤ │   │
│                       │  │    Memory Management    │ │   │
│                       │  │    - Heap               │ │   │
│                       │  │    - Stack              │ │   │
│                       │  │    - Method Area        │ │   │
│                       │  │    - Garbage Collector  │ │   │
│                       │  └─────────────────────────┘ │   │
│                       └─────────────────────────────────┘   │
├─────────────────────────────────────────────────────────┤
│                    Operating System                     │
└─────────────────────────────────────────────────────────┘
```

### **Java Ecosystem Components**

#### **Development Tools**
- **javac** - Java Compiler
- **java** - Java Runtime
- **jdb** - Java Debugger
- **jstack, jmap, jstat** - Profiling tools

#### **Build Tools**
- **Maven** - Project management and build automation
- **Gradle** - Modern build automation tool
- **Ant** - Legacy build tool

#### **IDE Support**
- **IntelliJ IDEA** - Most popular for Spring development
- **Eclipse** - Open-source IDE
- **Visual Studio Code** - Lightweight with Java extensions

---

## 🌟 Spring Framework Introduction

### **History and Evolution**

Spring Framework ถูกสร้างขึ้นโดย Rod Johnson ในปี 2003 เพื่อแก้ปัญหาความซับซ้อนของ Java Enterprise Edition (J2EE)

#### **Spring Timeline**
```
Spring 1.0 (2004) → Spring 2.0 (2006) → Spring 3.0 (2009) →
Spring 4.0 (2013) → Spring 5.0 (2017) → Spring 6.0 (2022)
```

#### **Spring Philosophy**
- 🎯 **Simplification** - ลดความซับซ้อนของ enterprise development
- 🔧 **POJO-based** - Plain Old Java Objects
- 🏗️ **Non-invasive** - ไม่ต้องการ inheritance หรือ implementation
- 🧩 **Modular** - สามารถใช้เฉพาะส่วนที่ต้องการ

### **Core Spring Framework Features**

#### **1. Inversion of Control (IoC)**
```java
// Traditional approach - tight coupling
public class OrderService {
    private PaymentService paymentService = new PaymentService(); // ❌ Hard dependency
    
    public void processOrder(Order order) {
        paymentService.processPayment(order.getTotal());
    }
}

// Spring IoC approach - loose coupling
@Service
public class OrderService {
    @Autowired
    private PaymentService paymentService; // ✅ Dependency injection
    
    public void processOrder(Order order) {
        paymentService.processPayment(order.getTotal());
    }
}
```

#### **2. Dependency Injection (DI)**
```java
// Constructor Injection (Recommended)
@Service
public class OrderService {
    private final PaymentService paymentService;
    private final InventoryService inventoryService;
    
    public OrderService(PaymentService paymentService, 
                       InventoryService inventoryService) {
        this.paymentService = paymentService;
        this.inventoryService = inventoryService;
    }
}

// Field Injection (Not recommended for testing)
@Service
public class OrderService {
    @Autowired
    private PaymentService paymentService;
}

// Setter Injection (For optional dependencies)
@Service
public class OrderService {
    private PaymentService paymentService;
    
    @Autowired
    public void setPaymentService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }
}
```

#### **3. Aspect-Oriented Programming (AOP)**
```java
@Aspect
@Component
public class LoggingAspect {
    
    @Before("@annotation(Loggable)")
    public void logMethodEntry(JoinPoint joinPoint) {
        log.info("Entering method: {}", joinPoint.getSignature().getName());
    }
    
    @AfterReturning(value = "@annotation(Loggable)", returning = "result")
    public void logMethodExit(JoinPoint joinPoint, Object result) {
        log.info("Exiting method: {} with result: {}", 
                joinPoint.getSignature().getName(), result);
    }
}

// Usage
@Service
public class OrderService {
    
    @Loggable
    public Order createOrder(OrderRequest request) {
        // Business logic
        return new Order();
    }
}
```

### **Spring Framework Modules**

```
┌─────────────────────────────────────────────────────────┐
│                    Spring Framework                     │
├─────────────────┬─────────────────┬─────────────────────┤
│   Core Module   │   Data Access   │   Web Module        │
│   - IoC         │   - JDBC        │   - MVC             │
│   - DI          │   - ORM         │   - WebFlux         │
│   - AOP         │   - JPA         │   - WebSocket       │
│   - Events      │   - Transactions│   - REST            │
├─────────────────┼─────────────────┼─────────────────────┤
│  Security       │   Integration   │   Testing           │
│  - Auth         │   - JMS         │   - Unit Tests      │
│  - Authorization│   - Email       │   - Integration     │
│  - OAuth        │   - Scheduling  │   - MockMVC         │
└─────────────────┴─────────────────┴─────────────────────┘
```

#### **Core Spring Modules**
- **spring-core** - IoC and DI functionality
- **spring-context** - Application context and configuration
- **spring-beans** - Bean factory and configuration
- **spring-aop** - Aspect-oriented programming

#### **Web Modules**
- **spring-web** - Basic web functionality
- **spring-webmvc** - Model-View-Controller framework
- **spring-webflux** - Reactive web framework

#### **Data Access Modules**
- **spring-jdbc** - JDBC abstraction layer
- **spring-orm** - Object-relational mapping support
- **spring-tx** - Transaction management

---

## 🚀 Spring Boot Introduction

### **What is Spring Boot?**

Spring Boot เป็น framework ที่สร้างขึ้นบน Spring Framework เพื่อ**ลดความซับซ้อน**และ**เร่งการพัฒนา** enterprise applications

#### **Spring Boot Goals**
- 🚀 **Rapid Development** - ลดเวลาการ setup และ configuration
- ⚙️ **Convention over Configuration** - ใช้ default configuration ที่ดี
- 📦 **Embedded Servers** - ไม่ต้องติดตั้ง external server
- 🔧 **Production-ready** - มี monitoring และ management built-in

### **Spring vs Spring Boot Comparison**

| Aspect | Spring Framework | Spring Boot |
|--------|------------------|-------------|
| **Configuration** | XML/Annotation ขนาดใหญ่ | Auto-configuration |
| **Dependency Management** | Manual dependency resolution | Starter dependencies |
| **Server Deployment** | External server (Tomcat/Jetty) | Embedded server |
| **Setup Time** | Hours to days | Minutes |
| **Production Features** | Manual setup | Built-in (Actuator) |
| **Learning Curve** | Steep | Gentle |

### **Spring Boot Key Features**

#### **1. Auto-Configuration**
```java
// Spring Boot automatically configures based on classpath
@SpringBootApplication  // Combines @Configuration, @EnableAutoConfiguration, @ComponentScan
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}

// Automatic DataSource configuration when H2 is on classpath
// Automatic JPA configuration when spring-boot-starter-data-jpa is included
// Automatic Security configuration when spring-boot-starter-security is included
```

#### **2. Starter Dependencies**
```xml
<!-- Traditional Spring setup -->
<dependencies>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-core</artifactId>
        <version>5.3.21</version>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
        <version>5.3.21</version>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-web</artifactId>
        <version>5.3.21</version>
    </dependency>
    <!-- Many more dependencies... -->
</dependencies>

<!-- Spring Boot starter approach -->
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <!-- Everything needed for web development included! -->
</dependencies>
```

#### **3. Embedded Server**
```java
// Traditional deployment
// 1. Build WAR file
// 2. Install Tomcat server
// 3. Deploy WAR to server
// 4. Configure server settings

// Spring Boot approach
@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
        // Embedded Tomcat starts automatically!
        // Application runs on http://localhost:8080
    }
}
```

#### **4. Production-Ready Features**
```java
// Actuator endpoints automatically available
// http://localhost:8080/actuator/health
// http://localhost:8080/actuator/metrics
// http://localhost:8080/actuator/info
// http://localhost:8080/actuator/env

@RestController
public class HealthController {
    
    @GetMapping("/api/status")
    public Map<String, String> getStatus() {
        return Map.of(
            "status", "UP",
            "timestamp", Instant.now().toString()
        );
    }
}
```

### **Spring Boot Starters**

#### **Common Starters**
```xml
<!-- Web Development -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>

<!-- Data Access -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>

<!-- Security -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>

<!-- Testing -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>

<!-- Reactive Web -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-webflux</artifactId>
</dependency>

<!-- Actuator for monitoring -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

---

## 🛠️ Getting Started Example

### **Your First Spring Boot Application**

#### **1. Project Structure**
```
my-spring-boot-app/
├── src/
│   ├── main/
│   │   ├── java/
│   │   │   └── com/example/demo/
│   │   │       ├── DemoApplication.java
│   │   │       ├── controller/
│   │   │       │   └── HelloController.java
│   │   │       └── model/
│   │   │           └── Greeting.java
│   │   └── resources/
│   │       ├── application.properties
│   │       ├── static/
│   │       └── templates/
│   └── test/
│       └── java/
└── pom.xml
```

#### **2. Main Application**
```java
package com.example.demo;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class DemoApplication {
    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }
}
```

#### **3. REST Controller**
```java
package com.example.demo.controller;

import com.example.demo.model.Greeting;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/api")
public class HelloController {
    
    @GetMapping("/hello")
    public Greeting hello(@RequestParam(defaultValue = "World") String name) {
        return new Greeting("Hello, " + name + "!");
    }
    
    @PostMapping("/greet")
    public Greeting greet(@RequestBody GreetRequest request) {
        return new Greeting("Hello, " + request.getName() + "! Welcome to Spring Boot.");
    }
}
```

#### **4. Model Class**
```java
package com.example.demo.model;

public class Greeting {
    private String message;
    private long timestamp;
    
    public Greeting(String message) {
        this.message = message;
        this.timestamp = System.currentTimeMillis();
    }
    
    // Getters and setters
    public String getMessage() { return message; }
    public void setMessage(String message) { this.message = message; }
    public long getTimestamp() { return timestamp; }
    public void setTimestamp(long timestamp) { this.timestamp = timestamp; }
}

class GreetRequest {
    private String name;
    
    // Getters and setters
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
}
```

#### **5. Configuration**
```properties
# application.properties
server.port=8080
spring.application.name=demo-application

# Logging
logging.level.com.example.demo=DEBUG
logging.pattern.console=%d{yyyy-MM-dd HH:mm:ss} - %msg%n

# Actuator
management.endpoints.web.exposure.include=health,info,metrics
management.endpoint.health.show-details=always
```

#### **6. Maven Configuration**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0">
    <modelVersion>4.0.0</modelVersion>
    
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.2.0</version>
        <relativePath/>
    </parent>
    
    <groupId>com.example</groupId>
    <artifactId>demo</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>demo</name>
    <description>Demo project for Spring Boot</description>
    
    <properties>
        <java.version>17</java.version>
    </properties>
    
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
    
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```

### **Running the Application**

```bash
# Compile and run
mvn spring-boot:run

# Or compile to JAR and run
mvn clean package
java -jar target/demo-0.0.1-SNAPSHOT.jar

# Test the endpoints
curl http://localhost:8080/api/hello
curl http://localhost:8080/api/hello?name=John
curl -X POST http://localhost:8080/api/greet \
     -H "Content-Type: application/json" \
     -d '{"name":"Spring Boot"}'

# Check health
curl http://localhost:8080/actuator/health
```

---

## 🎯 Industry Usage and Ecosystem

### **Spring Boot in Enterprise**

#### **Popular Use Cases**
- 🌐 **Microservices Architecture** - Netflix, Amazon, Uber
- 🏦 **Financial Services** - Banks, trading platforms
- 🛒 **E-commerce Platforms** - Online retail systems
- 📱 **Mobile Backend Services** - API services for mobile apps
- 🎮 **Gaming Platforms** - Real-time multiplayer systems

#### **Major Companies Using Spring Boot**
- **Netflix** - Microservices architecture with Spring Cloud
- **Amazon** - Internal services and AWS offerings
- **Alibaba** - E-commerce platform services
- **Spotify** - Music streaming backend services
- **LinkedIn** - Professional networking platform

### **Spring Ecosystem Projects**

#### **Spring Cloud**
```java
// Service Discovery
@EnableEurekaClient
@SpringBootApplication
public class UserServiceApplication {
    public static void main(String[] args) {
        SpringApplication.run(UserServiceApplication.class, args);
    }
}

// API Gateway
@EnableZuulProxy
@SpringBootApplication
public class GatewayApplication {
    public static void main(String[] args) {
        SpringApplication.run(GatewayApplication.class, args);
    }
}
```

#### **Spring Data**
```java
// Repository pattern with Spring Data JPA
@Repository
public interface UserRepository extends JpaRepository<User, Long> {
    List<User> findByLastName(String lastName);
    
    @Query("SELECT u FROM User u WHERE u.email = ?1")
    Optional<User> findByEmail(String email);
}

// Usage
@Service
public class UserService {
    @Autowired
    private UserRepository userRepository;
    
    public List<User> findUsers(String lastName) {
        return userRepository.findByLastName(lastName);
    }
}
```

#### **Spring Security**
```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {
    
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(authz -> authz
                .requestMatchers("/api/public/**").permitAll()
                .requestMatchers("/api/admin/**").hasRole("ADMIN")
                .anyRequest().authenticated()
            )
            .oauth2Login(oauth2 -> oauth2
                .loginPage("/login")
                .defaultSuccessUrl("/dashboard")
            );
        return http.build();
    }
}
```

---

## 🔄 Development Workflow

### **Typical Spring Boot Development Process**

#### **1. Project Initialization**
```bash
# Using Spring Initializr (https://start.spring.io/)
# Or using Spring Boot CLI
spring init --dependencies=web,data-jpa,security my-project

# Or using IDE plugins
# IntelliJ IDEA: File → New → Project → Spring Initializr
# Eclipse: File → New → Spring Starter Project
```

#### **2. Development Cycle**
```bash
# Development workflow
1. Write code
2. Run tests (mvn test)
3. Run application (mvn spring-boot:run)
4. Test endpoints (Postman/curl)
5. Iterate
```

#### **3. Build and Deploy**
```bash
# Build production JAR
mvn clean package -Pprod

# Run with different profiles
java -jar -Dspring.profiles.active=prod target/app.jar

# Docker deployment
docker build -t my-spring-app .
docker run -p 8080:8080 my-spring-app
```

### **Best Practices from Day One**

#### **Project Structure**
```java
// ✅ Good structure
com.company.app/
├── Application.java              // Main class
├── config/                      // Configuration classes
├── controller/                  // REST controllers
├── service/                     // Business logic
├── repository/                  // Data access
├── domain/                      // Entity classes
├── dto/                        // Data transfer objects
├── exception/                  // Custom exceptions
└── util/                       // Utility classes

// ❌ Avoid flat structure
com.company.app/
├── Application.java
├── UserController.java
├── UserService.java
├── UserRepository.java
├── OrderController.java
└── ... (all classes in one package)
```

#### **Configuration Management**
```yaml
# application.yml - structured configuration
spring:
  application:
    name: my-spring-boot-app
  
  datasource:
    url: jdbc:h2:mem:testdb
    driver-class-name: org.h2.Driver
    username: sa
    password: password
    
  jpa:
    hibernate:
      ddl-auto: create-drop
    show-sql: true
    
  profiles:
    active: development

logging:
  level:
    com.company.app: DEBUG
    org.springframework: INFO
    
management:
  endpoints:
    web:
      exposure:
        include: health,info,metrics
```

---

## 📚 Summary and Next Steps

### **Key Takeaways**

#### **Java Platform**
- ✅ Platform independence through JVM
- ✅ Rich ecosystem with extensive libraries
- ✅ Strong typing and memory management
- ✅ Enterprise-grade features and performance

#### **Spring Framework**
- ✅ IoC and DI for loose coupling
- ✅ AOP for cross-cutting concerns
- ✅ Comprehensive enterprise features
- ✅ Modular architecture

#### **Spring Boot**
- ✅ Convention over configuration
- ✅ Auto-configuration reduces boilerplate
- ✅ Embedded servers for easy deployment
- ✅ Production-ready features built-in

### **What's Next?**

#### **Chapter 2: Development Environment**
- IntelliJ IDEA setup and configuration
- Maven/Gradle project management
- Git version control best practices
- Docker containerization basics

#### **Chapter 3: Spring Boot Fundamentals**
- Deep dive into auto-configuration
- Understanding starter dependencies
- Configuration properties and profiles
- Application lifecycle and events

#### **Preparation for Next Chapter**
- Install IntelliJ IDEA Community/Ultimate
- Install Java 17 or later
- Set up Maven (3.8+)
- Create GitHub account if you don't have one

---

## 🏋️ Practice Exercises

### **Exercise 1: Environment Check**
```bash
# Verify your Java installation
java -version
javac -version

# Check Maven installation
mvn -version

# Test basic Java compilation
echo 'public class Hello { public static void main(String[] args) { System.out.println("Hello, Java!"); } }' > Hello.java
javac Hello.java
java Hello
```

### **Exercise 2: Spring Initializr**
1. Visit https://start.spring.io/
2. Create a project with:
   - **Project**: Maven
   - **Language**: Java
   - **Spring Boot**: 3.2.x (latest stable)
   - **Dependencies**: Spring Web, Spring Boot Actuator
3. Download and extract the project
4. Import into your IDE
5. Run the application
6. Test the actuator endpoints

### **Exercise 3: First REST API**
Create a simple REST API with the following endpoints:
- `GET /api/time` - Returns current timestamp
- `GET /api/random` - Returns a random number
- `POST /api/echo` - Returns the request body as response

### **Exercise 4: Dependency Analysis**
1. Examine the dependencies in your `pom.xml`
2. Run `mvn dependency:tree` to see the dependency graph
3. Identify which Spring modules are included
4. Note the version numbers and their relationships

---

## 🔗 Additional Resources

### **Official Documentation**
- [Spring Framework Documentation](https://docs.spring.io/spring-framework/docs/current/reference/html/)
- [Spring Boot Documentation](https://docs.spring.io/spring-boot/docs/current/reference/html/)
- [Spring Guides](https://spring.io/guides)

### **Community Resources**
- [Stack Overflow - Spring Boot](https://stackoverflow.com/questions/tagged/spring-boot)
- [Spring Community Forums](https://community.spring.io/)
- [Baeldung Spring Tutorials](https://www.baeldung.com/spring-tutorial)

### **Learning Tools**
- [Spring Boot CLI](https://docs.spring.io/spring-boot/docs/current/reference/html/cli.html)
- [Spring Tool Suite (STS)](https://spring.io/tools)
- [IntelliJ IDEA Spring Plugin](https://plugins.jetbrains.com/plugin/20221-spring-boot-helper)

---

**🎯 Ready for the next chapter? Let's set up your development environment!**

**[Next Chapter: Development Environment Setup →](./02-development-environment.md)**

---

*Duration: ~3 hours | Difficulty: Beginner | Prerequisites: Basic Java knowledge*
