# Chapter 4: Spring Core Concepts
## IoC, Dependency Injection, Beans, และ Configuration ใน Spring Framework

### 🎯 Learning Objectives
หลังจากเรียนบทนี้แล้ว คุณจะสามารถ:
- เข้าใจหลักการ Inversion of Control (IoC) และ Dependency Injection (DI)
- จัดการ Bean Definition และ Bean Scopes อย่างเชี่ยวชาญ
- ใช้ Configuration แบบต่างๆ (Annotation, Java Config, XML)
- เข้าใจ Bean Lifecycle และ Post-Processors
- ประยุกต์ใช้ Advanced DI patterns ในโปรเจ็คจริง

---

## 🔄 Inversion of Control (IoC) Deep Dive

### **Understanding IoC Principle**

IoC เป็นหลักการที่**กลับด้านการควบคุม**จากการที่ object สร้าง dependencies ของตัวเอง มาเป็น external container เป็นผู้จัดการแทน

#### **Traditional Approach vs IoC**
```java
// ❌ Traditional Approach - Tight Coupling
public class OrderService {
    private PaymentService paymentService;
    private InventoryService inventoryService;
    private EmailService emailService;
    
    public OrderService() {
        // Object creates its own dependencies
        this.paymentService = new CreditCardPaymentService();
        this.inventoryService = new DatabaseInventoryService();
        this.emailService = new SmtpEmailService();
    }
    
    public void processOrder(Order order) {
        // Hard to test, hard to change implementation
        inventoryService.reserveItems(order.getItems());
        paymentService.processPayment(order.getTotal());
        emailService.sendConfirmation(order.getCustomerEmail());
    }
}

// ✅ IoC Approach - Loose Coupling
@Service
public class OrderService {
    private final PaymentService paymentService;
    private final InventoryService inventoryService;
    private final EmailService emailService;
    
    // Dependencies injected by IoC container
    public OrderService(PaymentService paymentService,
                       InventoryService inventoryService,
                       EmailService emailService) {
        this.paymentService = paymentService;
        this.inventoryService = inventoryService;
        this.emailService = emailService;
    }
    
    public void processOrder(Order order) {
        // Easy to test, easy to change implementations
        inventoryService.reserveItems(order.getItems());
        paymentService.processPayment(order.getTotal());
        emailService.sendConfirmation(order.getCustomerEmail());
    }
}
```

### **Benefits of IoC**

#### **1. Loose Coupling**
```java
// Interface-based programming
public interface PaymentService {
    PaymentResult processPayment(BigDecimal amount, PaymentMethod method);
}

@Service("creditCardPayment")
public class CreditCardPaymentService implements PaymentService {
    @Override
    public PaymentResult processPayment(BigDecimal amount, PaymentMethod method) {
        // Credit card processing logic
        return new PaymentResult(true, "CC-" + UUID.randomUUID());
    }
}

@Service("paypalPayment")
public class PayPalPaymentService implements PaymentService {
    @Override
    public PaymentResult processPayment(BigDecimal amount, PaymentMethod method) {
        // PayPal processing logic
        return new PaymentResult(true, "PP-" + UUID.randomUUID());
    }
}

// Consumer doesn't know which implementation is used
@Service
public class OrderService {
    private final PaymentService paymentService;
    
    public OrderService(@Qualifier("creditCardPayment") PaymentService paymentService) {
        this.paymentService = paymentService;
    }
}
```

#### **2. Testability**
```java
// Easy to test with mock dependencies
@ExtendWith(MockitoExtension.class)
class OrderServiceTest {
    
    @Mock
    private PaymentService paymentService;
    
    @Mock
    private InventoryService inventoryService;
    
    @Mock
    private EmailService emailService;
    
    @InjectMocks
    private OrderService orderService;
    
    @Test
    void shouldProcessOrderSuccessfully() {
        // Given
        Order order = new Order("customer@example.com", BigDecimal.valueOf(100));
        when(inventoryService.reserveItems(any())).thenReturn(true);
        when(paymentService.processPayment(any(), any()))
            .thenReturn(new PaymentResult(true, "payment-id"));
        
        // When
        OrderResult result = orderService.processOrder(order);
        
        // Then
        assertThat(result.isSuccess()).isTrue();
        verify(emailService).sendConfirmation("customer@example.com");
    }
}
```

#### **3. Configuration Flexibility**
```java
@Configuration
public class PaymentConfiguration {
    
    @Bean
    @Primary
    @ConditionalOnProperty(name = "payment.provider", havingValue = "stripe")
    public PaymentService stripePaymentService() {
        return new StripePaymentService();
    }
    
    @Bean
    @ConditionalOnProperty(name = "payment.provider", havingValue = "paypal")
    public PaymentService paypalPaymentService() {
        return new PayPalPaymentService();
    }
    
    @Bean
    @ConditionalOnProperty(name = "payment.provider", havingValue = "mock", matchIfMissing = true)
    public PaymentService mockPaymentService() {
        return new MockPaymentService();
    }
}
```

---

## 💉 Dependency Injection (DI) Patterns

### **Constructor Injection (Recommended)**

#### **Why Constructor Injection is Preferred**
```java
@Service
public class UserService {
    private final UserRepository userRepository;
    private final PasswordEncoder passwordEncoder;
    private final EmailService emailService;
    
    // ✅ Constructor injection - Recommended
    public UserService(UserRepository userRepository,
                      PasswordEncoder passwordEncoder,
                      EmailService emailService) {
        this.userRepository = userRepository;
        this.passwordEncoder = passwordEncoder;
        this.emailService = emailService;
    }
    
    // Benefits:
    // 1. Immutable dependencies (final fields)
    // 2. Fail-fast if dependencies missing
    // 3. Easy to test
    // 4. No need for @Autowired annotation
}
```

#### **Optional Dependencies with Constructor Injection**
```java
@Service
public class NotificationService {
    private final EmailService emailService;
    private final SmsService smsService;
    private final PushNotificationService pushService;
    
    public NotificationService(EmailService emailService,
                             @Autowired(required = false) SmsService smsService,
                             Optional<PushNotificationService> pushService) {
        this.emailService = emailService;
        this.smsService = smsService;
        this.pushService = pushService.orElse(null);
    }
    
    public void sendNotification(String message, User user) {
        // Email is mandatory
        emailService.send(message, user.getEmail());
        
        // SMS is optional
        if (smsService != null && user.getPhoneNumber() != null) {
            smsService.send(message, user.getPhoneNumber());
        }
        
        // Push notification is optional
        if (pushService != null && user.getPushToken() != null) {
            pushService.send(message, user.getPushToken());
        }
    }
}
```

### **Field Injection (Not Recommended for Production)**

#### **Field Injection Example**
```java
@Service
public class OrderService {
    
    // ❌ Field injection - Avoid in production
    @Autowired
    private PaymentService paymentService;
    
    @Autowired
    private InventoryService inventoryService;
    
    // Problems:
    // 1. Cannot make fields final
    // 2. Hard to test (need reflection or Spring context)
    // 3. Hidden dependencies
    // 4. Potential circular dependencies
    // 5. Violates immutability principle
}
```

#### **When Field Injection Might Be Acceptable**
```java
@Component
public class DevelopmentDataInitializer {
    
    // Acceptable for simple utility components
    @Autowired
    private UserRepository userRepository;
    
    @PostConstruct
    public void initializeTestData() {
        if (userRepository.count() == 0) {
            // Initialize test data
        }
    }
}
```

### **Setter Injection (For Optional Dependencies)**

#### **Setter Injection Example**
```java
@Service
public class ReportService {
    private ReportRepository reportRepository;
    private CacheService cacheService;  // Optional
    private MetricsService metricsService;  // Optional
    
    // Required dependency
    @Autowired
    public ReportService(ReportRepository reportRepository) {
        this.reportRepository = reportRepository;
    }
    
    // Optional dependency
    @Autowired(required = false)
    public void setCacheService(CacheService cacheService) {
        this.cacheService = cacheService;
    }
    
    // Optional dependency
    @Autowired(required = false)
    public void setMetricsService(MetricsService metricsService) {
        this.metricsService = metricsService;
    }
    
    public Report generateReport(String reportId) {
        Report report;
        
        // Try cache first if available
        if (cacheService != null) {
            report = cacheService.get(reportId);
            if (report != null) {
                return report;
            }
        }
        
        // Generate report
        report = reportRepository.generateReport(reportId);
        
        // Cache if available
        if (cacheService != null) {
            cacheService.put(reportId, report);
        }
        
        // Record metrics if available
        if (metricsService != null) {
            metricsService.recordReportGeneration(reportId);
        }
        
        return report;
    }
}
```

### **Method Injection (Advanced)**

#### **Lookup Method Injection**
```java
@Component
@Scope("singleton")
public abstract class CommandProcessor {
    
    // Singleton bean needs to get prototype instances
    @Lookup
    public abstract Command createCommand();
    
    public void processCommands(List<String> inputs) {
        for (String input : inputs) {
            Command command = createCommand();  // New instance each time
            command.setInput(input);
            command.execute();
        }
    }
}

@Component
@Scope("prototype")
public class Command {
    private String input;
    
    public void setInput(String input) {
        this.input = input;
    }
    
    public void execute() {
        System.out.println("Executing command with input: " + input);
    }
}
```

---

## 🫘 Bean Definition and Management

### **Bean Definition Methods**

#### **Annotation-Based Bean Definition**
```java
// Component stereotypes
@Component    // Generic component
@Service      // Service layer
@Repository   // Data access layer
@Controller   // Web layer
@RestController // REST web service

// Custom component
@Service("userService")
public class UserServiceImpl implements UserService {
    // Implementation
}

// Repository with custom name
@Repository("jpaUserRepository")
public class JpaUserRepository implements UserRepository {
    // Implementation
}
```

#### **Java Configuration-Based Bean Definition**
```java
@Configuration
public class ApplicationConfiguration {
    
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder(12);
    }
    
    @Bean("userValidator")
    public Validator<User> userValidator() {
        return new UserValidator();
    }
    
    @Bean
    @Primary
    public DataSource primaryDataSource() {
        HikariConfig config = new HikariConfig();
        config.setJdbcUrl("jdbc:postgresql://localhost/primary");
        config.setUsername("user");
        config.setPassword("password");
        return new HikariDataSource(config);
    }
    
    @Bean("secondaryDataSource")
    public DataSource secondaryDataSource() {
        HikariConfig config = new HikariConfig();
        config.setJdbcUrl("jdbc:postgresql://localhost/secondary");
        config.setUsername("user");
        config.setPassword("password");
        return new HikariDataSource(config);
    }
    
    @Bean
    public UserService userService(UserRepository userRepository, 
                                  PasswordEncoder passwordEncoder) {
        return new UserServiceImpl(userRepository, passwordEncoder);
    }
}
```

#### **Conditional Bean Creation**
```java
@Configuration
public class ConditionalConfiguration {
    
    @Bean
    @ConditionalOnProperty(name = "cache.enabled", havingValue = "true")
    @ConditionalOnClass(RedisTemplate.class)
    public CacheService redisCacheService() {
        return new RedisCacheService();
    }
    
    @Bean
    @ConditionalOnMissingBean(CacheService.class)
    public CacheService inMemoryCacheService() {
        return new InMemoryCacheService();
    }
    
    @Bean
    @Profile("development")
    public DataInitializer developmentDataInitializer() {
        return new DevelopmentDataInitializer();
    }
    
    @Bean
    @Profile("production")
    public DataInitializer productionDataInitializer() {
        return new ProductionDataInitializer();
    }
}
```

### **Bean Scopes Deep Dive**

#### **Singleton Scope (Default)**
```java
@Service
@Scope("singleton")  // Default, can be omitted
public class ConfigurationService {
    private final Map<String, String> configurations = new ConcurrentHashMap<>();
    
    // One instance per ApplicationContext
    // Shared across all requests
    // Must be thread-safe for concurrent access
    
    public void setConfiguration(String key, String value) {
        configurations.put(key, value);
    }
    
    public String getConfiguration(String key) {
        return configurations.get(key);
    }
}
```

#### **Prototype Scope**
```java
@Component
@Scope("prototype")
public class TaskProcessor {
    private String taskId;
    private long startTime;
    
    // New instance created every time bean is requested
    // Not shared between requests
    // Spring doesn't manage destruction
    
    @PostConstruct
    public void initialize() {
        this.taskId = UUID.randomUUID().toString();
        this.startTime = System.currentTimeMillis();
    }
    
    public void processTask(Task task) {
        // Process task with unique instance
    }
}

// Using prototype beans
@Service
public class TaskManager {
    private final ApplicationContext applicationContext;
    
    public TaskManager(ApplicationContext applicationContext) {
        this.applicationContext = applicationContext;
    }
    
    public void processTasks(List<Task> tasks) {
        for (Task task : tasks) {
            TaskProcessor processor = applicationContext.getBean(TaskProcessor.class);
            processor.processTask(task);
        }
    }
}
```

#### **Web Scopes (Request, Session, Application)**
```java
// Request scope - new instance per HTTP request
@Component
@Scope(WebApplicationContext.SCOPE_REQUEST)
public class RequestContext {
    private String requestId;
    private LocalDateTime timestamp;
    
    @PostConstruct
    public void initialize() {
        this.requestId = UUID.randomUUID().toString();
        this.timestamp = LocalDateTime.now();
    }
    
    // Getters
    public String getRequestId() { return requestId; }
    public LocalDateTime getTimestamp() { return timestamp; }
}

// Session scope - new instance per HTTP session
@Component
@Scope(WebApplicationContext.SCOPE_SESSION)
public class UserSession {
    private User currentUser;
    private Set<String> permissions = new HashSet<>();
    private LocalDateTime loginTime;
    
    public void login(User user) {
        this.currentUser = user;
        this.loginTime = LocalDateTime.now();
        this.permissions = loadUserPermissions(user);
    }
    
    // Methods to manage session data
}

// Application scope - one instance per ServletContext
@Component
@Scope(WebApplicationContext.SCOPE_APPLICATION)
public class ApplicationMetrics {
    private final AtomicLong requestCount = new AtomicLong(0);
    private final AtomicLong errorCount = new AtomicLong(0);
    private final LocalDateTime startTime = LocalDateTime.now();
    
    public void incrementRequestCount() {
        requestCount.incrementAndGet();
    }
    
    public void incrementErrorCount() {
        errorCount.incrementAndGet();
    }
    
    // Getters for metrics
}
```

#### **Custom Scope**
```java
// Custom scope implementation
public class ThreadScope implements Scope {
    private final ThreadLocal<Map<String, Object>> threadStorage = new ThreadLocal<>();
    
    @Override
    public Object get(String name, ObjectFactory<?> objectFactory) {
        Map<String, Object> scope = threadStorage.get();
        if (scope == null) {
            scope = new HashMap<>();
            threadStorage.set(scope);
        }
        
        Object object = scope.get(name);
        if (object == null) {
            object = objectFactory.getObject();
            scope.put(name, object);
        }
        return object;
    }
    
    @Override
    public Object remove(String name) {
        Map<String, Object> scope = threadStorage.get();
        if (scope != null) {
            return scope.remove(name);
        }
        return null;
    }
    
    @Override
    public void registerDestructionCallback(String name, Runnable callback) {
        // Register cleanup callback
    }
    
    @Override
    public Object resolveContextualObject(String key) {
        return null;
    }
    
    @Override
    public String getConversationId() {
        return Thread.currentThread().getName();
    }
}

// Register custom scope
@Configuration
public class ScopeConfiguration {
    
    @Bean
    public static BeanFactoryPostProcessor beanFactoryPostProcessor() {
        return beanFactory -> {
            beanFactory.registerScope("thread", new ThreadScope());
        };
    }
}

// Use custom scope
@Component
@Scope("thread")
public class ThreadLocalService {
    private String threadId = Thread.currentThread().getName();
    
    public String getThreadId() {
        return threadId;
    }
}
```

---

## 🏗️ Configuration Approaches

### **Annotation-Based Configuration**

#### **Component Scanning**
```java
@SpringBootApplication
@ComponentScan(
    basePackages = {"com.example.service", "com.example.repository"},
    includeFilters = @ComponentScan.Filter(type = FilterType.ANNOTATION, classes = Service.class),
    excludeFilters = @ComponentScan.Filter(type = FilterType.ANNOTATION, classes = Controller.class)
)
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}

// Custom component filter
public class CustomTypeFilter implements TypeFilter {
    @Override
    public boolean match(MetadataReader metadataReader, 
                        MetadataReaderFactory metadataReaderFactory) throws IOException {
        String className = metadataReader.getClassMetadata().getClassName();
        return className.endsWith("ServiceImpl");
    }
}

@ComponentScan(
    includeFilters = @ComponentScan.Filter(type = FilterType.CUSTOM, classes = CustomTypeFilter.class)
)
public class CustomScanConfiguration {
}
```

#### **Qualifier and Primary Annotations**
```java
// Multiple implementations
public interface NotificationService {
    void sendNotification(String message, String recipient);
}

@Service("emailNotification")
public class EmailNotificationService implements NotificationService {
    @Override
    public void sendNotification(String message, String recipient) {
        // Send email
    }
}

@Service("smsNotification")
public class SmsNotificationService implements NotificationService {
    @Override
    public void sendNotification(String message, String recipient) {
        // Send SMS
    }
}

@Service
@Primary  // Default choice when multiple implementations exist
public class PushNotificationService implements NotificationService {
    @Override
    public void sendNotification(String message, String recipient) {
        // Send push notification
    }
}

// Using specific implementation
@Service
public class OrderService {
    private final NotificationService emailService;
    private final NotificationService smsService;
    private final NotificationService defaultService;
    
    public OrderService(@Qualifier("emailNotification") NotificationService emailService,
                       @Qualifier("smsNotification") NotificationService smsService,
                       NotificationService defaultService) {  // Injects @Primary bean
        this.emailService = emailService;
        this.smsService = smsService;
        this.defaultService = defaultService;
    }
}
```

#### **Custom Qualifier Annotations**
```java
// Custom qualifier annotation
@Target({ElementType.FIELD, ElementType.PARAMETER, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Qualifier
public @interface NotificationType {
    String value();
}

// Use custom qualifier
@Service
@NotificationType("email")
public class EmailNotificationService implements NotificationService {
    // Implementation
}

@Service
@NotificationType("sms")
public class SmsNotificationService implements NotificationService {
    // Implementation
}

// Inject with custom qualifier
@Service
public class NotificationManager {
    private final NotificationService emailService;
    private final NotificationService smsService;
    
    public NotificationManager(@NotificationType("email") NotificationService emailService,
                              @NotificationType("sms") NotificationService smsService) {
        this.emailService = emailService;
        this.smsService = smsService;
    }
}
```

### **Java Configuration (Recommended)**

#### **Configuration Classes**
```java
@Configuration
@EnableJpaRepositories(basePackages = "com.example.repository")
@EnableScheduling
@EnableAsync
public class ApplicationConfiguration {
    
    @Bean
    @ConfigurationProperties(prefix = "app.datasource")
    public DataSourceProperties dataSourceProperties() {
        return new DataSourceProperties();
    }
    
    @Bean
    @Primary
    public DataSource dataSource(DataSourceProperties properties) {
        return properties.initializeDataSourceBuilder()
                .type(HikariDataSource.class)
                .build();
    }
    
    @Bean
    public PlatformTransactionManager transactionManager(DataSource dataSource) {
        return new DataSourceTransactionManager(dataSource);
    }
    
    @Bean
    public TaskExecutor taskExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(5);
        executor.setMaxPoolSize(10);
        executor.setQueueCapacity(25);
        executor.setThreadNamePrefix("async-");
        executor.initialize();
        return executor;
    }
}
```

#### **Modular Configuration**
```java
// Database configuration
@Configuration
@Profile("!test")
public class DatabaseConfiguration {
    
    @Bean
    @Primary
    public DataSource primaryDataSource() {
        // Primary database configuration
    }
    
    @Bean("readOnlyDataSource")
    public DataSource readOnlyDataSource() {
        // Read-only replica configuration
    }
    
    @Bean
    public JdbcTemplate jdbcTemplate(@Qualifier("primaryDataSource") DataSource dataSource) {
        return new JdbcTemplate(dataSource);
    }
}

// Security configuration
@Configuration
@EnableWebSecurity
public class SecurityConfiguration {
    
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder(12);
    }
    
    @Bean
    public AuthenticationManager authenticationManager(
            AuthenticationConfiguration authConfig) throws Exception {
        return authConfig.getAuthenticationManager();
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
                .allowedMethods("GET", "POST", "PUT", "DELETE", "OPTIONS")
                .allowCredentials(true)
                .maxAge(3600);
    }
    
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new LoggingInterceptor())
                .addPathPatterns("/api/**")
                .excludePathPatterns("/api/health");
    }
}
```

#### **Conditional Configuration**
```java
@Configuration
public class ConditionalBeansConfiguration {
    
    @Bean
    @ConditionalOnProperty(name = "feature.advanced-security", havingValue = "true")
    public SecurityEnhancer securityEnhancer() {
        return new AdvancedSecurityEnhancer();
    }
    
    @Bean
    @ConditionalOnMissingBean(SecurityEnhancer.class)
    public SecurityEnhancer basicSecurityEnhancer() {
        return new BasicSecurityEnhancer();
    }
    
    @Bean
    @ConditionalOnClass(name = "redis.clients.jedis.Jedis")
    @ConditionalOnProperty(name = "cache.type", havingValue = "redis")
    public CacheManager redisCacheManager() {
        return new RedisCacheManager();
    }
    
    @Bean
    @ConditionalOnMissingBean(CacheManager.class)
    public CacheManager simpleCacheManager() {
        return new ConcurrentMapCacheManager();
    }
}
```

---

## 🔄 Bean Lifecycle and Post-Processors

### **Bean Lifecycle Phases**

#### **Complete Bean Lifecycle**
```java
@Component
public class LifecycleBean implements BeanNameAware, BeanFactoryAware, 
                                    ApplicationContextAware, InitializingBean, DisposableBean {
    
    private static final Logger log = LoggerFactory.getLogger(LifecycleBean.class);
    
    private String beanName;
    private BeanFactory beanFactory;
    private ApplicationContext applicationContext;
    
    // 1. Constructor
    public LifecycleBean() {
        log.info("1. Constructor called");
    }
    
    // 2. Set bean name
    @Override
    public void setBeanName(String name) {
        this.beanName = name;
        log.info("2. setBeanName: {}", name);
    }
    
    // 3. Set bean factory
    @Override
    public void setBeanFactory(BeanFactory beanFactory) throws BeansException {
        this.beanFactory = beanFactory;
        log.info("3. setBeanFactory called");
    }
    
    // 4. Set application context
    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        this.applicationContext = applicationContext;
        log.info("4. setApplicationContext called");
    }
    
    // 5. Custom initialization
    @PostConstruct
    public void postConstruct() {
        log.info("5. @PostConstruct called");
    }
    
    // 6. afterPropertiesSet from InitializingBean
    @Override
    public void afterPropertiesSet() throws Exception {
        log.info("6. afterPropertiesSet called");
    }
    
    // 7. Custom init method (if configured)
    public void customInit() {
        log.info("7. customInit called");
    }
    
    // Business methods
    public void doWork() {
        log.info("Doing business work...");
    }
    
    // Destruction phase
    @PreDestroy
    public void preDestroy() {
        log.info("8. @PreDestroy called");
    }
    
    @Override
    public void destroy() throws Exception {
        log.info("9. destroy() called");
    }
    
    public void customDestroy() {
        log.info("10. customDestroy called");
    }
}
```

#### **Configuration for Lifecycle Methods**
```java
@Configuration
public class LifecycleConfiguration {
    
    @Bean(initMethod = "customInit", destroyMethod = "customDestroy")
    public LifecycleBean lifecycleBean() {
        return new LifecycleBean();
    }
}
```

### **Bean Post Processors**

#### **BeanPostProcessor Implementation**
```java
@Component
public class LoggingBeanPostProcessor implements BeanPostProcessor {
    
    private static final Logger log = LoggerFactory.getLogger(LoggingBeanPostProcessor.class);
    
    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) 
            throws BeansException {
        if (bean instanceof UserService) {
            log.info("Before initialization of bean: {}", beanName);
        }
        return bean;
    }
    
    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) 
            throws BeansException {
        if (bean instanceof UserService) {
            log.info("After initialization of bean: {}", beanName);
            
            // Wrap bean with proxy if needed
            return createProxy(bean);
        }
        return bean;
    }
    
    private Object createProxy(Object bean) {
        return Proxy.newProxyInstance(
            bean.getClass().getClassLoader(),
            bean.getClass().getInterfaces(),
            (proxy, method, args) -> {
                log.info("Method {} called on {}", method.getName(), bean.getClass().getSimpleName());
                return method.invoke(bean, args);
            }
        );
    }
}
```

#### **BeanFactoryPostProcessor Implementation**
```java
@Component
public class PropertyPlaceholderBeanFactoryPostProcessor implements BeanFactoryPostProcessor {
    
    @Override
    public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) 
            throws BeansException {
        
        // Modify bean definitions before beans are created
        String[] beanNames = beanFactory.getBeanDefinitionNames();
        
        for (String beanName : beanNames) {
            BeanDefinition beanDefinition = beanFactory.getBeanDefinition(beanName);
            
            if (beanDefinition instanceof AbstractBeanDefinition) {
                AbstractBeanDefinition abd = (AbstractBeanDefinition) beanDefinition;
                
                // Modify scope based on conditions
                if (beanName.endsWith("Service") && abd.getScope().equals("singleton")) {
                    log.info("Modifying scope for service bean: {}", beanName);
                    abd.setScope("prototype");
                }
            }
        }
    }
}
```

#### **Custom Annotation Processing**
```java
// Custom annotation
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
public @interface Monitored {
    String value() default "";
}

// Annotation processor
@Component
public class MonitoredAnnotationProcessor implements BeanPostProcessor {
    
    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) 
            throws BeansException {
        
        Class<?> beanClass = bean.getClass();
        if (beanClass.isAnnotationPresent(Monitored.class)) {
            Monitored monitored = beanClass.getAnnotation(Monitored.class);
            
            return Proxy.newProxyInstance(
                beanClass.getClassLoader(),
                beanClass.getInterfaces(),
                new MonitoringInvocationHandler(bean, monitored.value())
            );
        }
        
        return bean;
    }
}

// Monitoring proxy
public class MonitoringInvocationHandler implements InvocationHandler {
    private final Object target;
    private final String monitoringName;
    private final MeterRegistry meterRegistry;
    
    public MonitoringInvocationHandler(Object target, String monitoringName) {
        this.target = target;
        this.monitoringName = monitoringName;
        this.meterRegistry = Metrics.globalRegistry;
    }
    
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        Timer.Sample sample = Timer.start(meterRegistry);
        
        try {
            Object result = method.invoke(target, args);
            sample.stop(Timer.builder("method.execution")
                .tag("class", target.getClass().getSimpleName())
                .tag("method", method.getName())
                .tag("status", "success")
                .register(meterRegistry));
            return result;
        } catch (Exception e) {
            sample.stop(Timer.builder("method.execution")
                .tag("class", target.getClass().getSimpleName())
                .tag("method", method.getName())
                .tag("status", "error")
                .register(meterRegistry));
            throw e;
        }
    }
}

// Usage
@Service
@Monitored("user-service")
public class UserService {
    public User findById(Long id) {
        // Implementation
    }
}
```

---

## 🔧 Advanced DI Patterns

### **Factory Beans**

#### **FactoryBean Implementation**
```java
// Custom factory bean
public class DatabaseConnectionFactoryBean implements FactoryBean<DataSource> {
    
    private String url;
    private String username;
    private String password;
    private String driverClassName;
    
    @Override
    public DataSource getObject() throws Exception {
        HikariConfig config = new HikariConfig();
        config.setJdbcUrl(url);
        config.setUsername(username);
        config.setPassword(password);
        config.setDriverClassName(driverClassName);
        
        // Custom connection pool configuration
        config.setMaximumPoolSize(20);
        config.setMinimumIdle(5);
        config.setConnectionTimeout(30000);
        config.setIdleTimeout(600000);
        config.setMaxLifetime(1800000);
        
        return new HikariDataSource(config);
    }
    
    @Override
    public Class<?> getObjectType() {
        return DataSource.class;
    }
    
    @Override
    public boolean isSingleton() {
        return true;
    }
    
    // Setters for properties
    public void setUrl(String url) { this.url = url; }
    public void setUsername(String username) { this.username = username; }
    public void setPassword(String password) { this.password = password; }
    public void setDriverClassName(String driverClassName) { this.driverClassName = driverClassName; }
}

// Configuration
@Configuration
public class FactoryBeanConfiguration {
    
    @Bean
    public DatabaseConnectionFactoryBean dataSourceFactory() {
        DatabaseConnectionFactoryBean factory = new DatabaseConnectionFactoryBean();
        factory.setUrl("jdbc:postgresql://localhost:5432/mydb");
        factory.setUsername("user");
        factory.setPassword("password");
        factory.setDriverClassName("org.postgresql.Driver");
        return factory;
    }
}
```

### **Circular Dependencies Resolution**

#### **Circular Dependency Example and Solutions**
```java
// ❌ Circular dependency problem
@Service
public class OrderService {
    private final UserService userService;
    
    public OrderService(UserService userService) {
        this.userService = userService;
    }
}

@Service
public class UserService {
    private final OrderService orderService;
    
    public UserService(OrderService orderService) {
        this.orderService = orderService;
    }
}

// ✅ Solution 1: Use @Lazy annotation
@Service
public class OrderService {
    private final UserService userService;
    
    public OrderService(@Lazy UserService userService) {
        this.userService = userService;
    }
}

// ✅ Solution 2: Use setter injection for one dependency
@Service
public class OrderService {
    private final UserService userService;
    private OrderHistoryService orderHistoryService;
    
    public OrderService(UserService userService) {
        this.userService = userService;
    }
    
    @Autowired
    public void setOrderHistoryService(OrderHistoryService orderHistoryService) {
        this.orderHistoryService = orderHistoryService;
    }
}

// ✅ Solution 3: Redesign to remove circular dependency
@Service
public class OrderService {
    private final UserRepository userRepository;
    
    public OrderService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
}

@Service
public class UserService {
    private final OrderRepository orderRepository;
    
    public UserService(OrderRepository orderRepository) {
        this.orderRepository = orderRepository;
    }
}
```

### **Bean Validation and Constraints**

#### **Bean Validation Integration**
```java
@Configuration
@EnableConfigurationProperties(AppProperties.class)
public class ValidationConfiguration {
    
    @Bean
    public Validator validator() {
        return Validation.buildDefaultValidatorFactory().getValidator();
    }
    
    @Bean
    public MethodValidationPostProcessor methodValidationPostProcessor() {
        MethodValidationPostProcessor processor = new MethodValidationPostProcessor();
        processor.setValidator(validator());
        return processor;
    }
}

// Properties with validation
@ConfigurationProperties(prefix = "app")
@Validated
public class AppProperties {
    
    @NotBlank
    @Size(min = 3, max = 50)
    private String name;
    
    @Email
    private String adminEmail;
    
    @Min(1)
    @Max(65535)
    private int serverPort = 8080;
    
    @Valid
    private Database database = new Database();
    
    public static class Database {
        @NotBlank
        private String url;
        
        @NotBlank
        private String username;
        
        @NotBlank
        private String password;
        
        @Min(1)
        @Max(100)
        private int maxConnections = 20;
        
        // Getters and setters
    }
    
    // Getters and setters
}

// Service with method validation
@Service
@Validated
public class UserService {
    
    public User createUser(@Valid @NotNull CreateUserRequest request) {
        // Implementation
    }
    
    public Optional<User> findById(@NotNull @Positive Long id) {
        // Implementation
    }
    
    public List<User> findByEmail(@NotBlank @Email String email) {
        // Implementation
    }
}
```

---

## 🧪 Practical Examples and Best Practices

### **Real-World Configuration Example**

#### **E-Commerce Application Configuration**
```java
// Main application configuration
@SpringBootApplication
@EnableJpaRepositories(basePackages = "com.example.ecommerce.repository")
@EnableScheduling
@EnableAsync
public class ECommerceApplication {
    public static void main(String[] args) {
        SpringApplication.run(ECommerceApplication.class, args);
    }
}

// Database configuration
@Configuration
@EnableTransactionManagement
public class DatabaseConfiguration {
    
    @Bean
    @Primary
    @ConfigurationProperties("app.datasource.primary")
    public DataSourceProperties primaryDataSourceProperties() {
        return new DataSourceProperties();
    }
    
    @Bean
    @Primary
    public DataSource primaryDataSource() {
        return primaryDataSourceProperties()
                .initializeDataSourceBuilder()
                .type(HikariDataSource.class)
                .build();
    }
    
    @Bean
    @ConfigurationProperties("app.datasource.readonly")
    public DataSourceProperties readOnlyDataSourceProperties() {
        return new DataSourceProperties();
    }
    
    @Bean("readOnlyDataSource")
    public DataSource readOnlyDataSource() {
        return readOnlyDataSourceProperties()
                .initializeDataSourceBuilder()
                .type(HikariDataSource.class)
                .build();
    }
    
    @Bean
    public JdbcTemplate primaryJdbcTemplate(@Qualifier("primaryDataSource") DataSource dataSource) {
        return new JdbcTemplate(dataSource);
    }
    
    @Bean("readOnlyJdbcTemplate")
    public JdbcTemplate readOnlyJdbcTemplate(@Qualifier("readOnlyDataSource") DataSource dataSource) {
        return new JdbcTemplate(dataSource);
    }
}

// Service layer configuration
@Configuration
public class ServiceConfiguration {
    
    @Bean
    @ConditionalOnProperty(name = "payment.provider", havingValue = "stripe")
    public PaymentService stripePaymentService(@Value("${stripe.api-key}") String apiKey) {
        return new StripePaymentService(apiKey);
    }
    
    @Bean
    @ConditionalOnProperty(name = "payment.provider", havingValue = "paypal")
    public PaymentService paypalPaymentService(@Value("${paypal.client-id}") String clientId,
                                             @Value("${paypal.client-secret}") String clientSecret) {
        return new PayPalPaymentService(clientId, clientSecret);
    }
    
    @Bean
    @ConditionalOnMissingBean(PaymentService.class)
    public PaymentService mockPaymentService() {
        return new MockPaymentService();
    }
    
    @Bean
    public EmailService emailService(@Value("${email.provider}") String provider) {
        return switch (provider.toLowerCase()) {
            case "smtp" -> new SmtpEmailService();
            case "sendgrid" -> new SendGridEmailService();
            case "ses" -> new AWSEmailService();
            default -> new MockEmailService();
        };
    }
}

// Cache configuration
@Configuration
@EnableCaching
@ConditionalOnProperty(name = "cache.enabled", havingValue = "true", matchIfMissing = true)
public class CacheConfiguration {
    
    @Bean
    @ConditionalOnProperty(name = "cache.type", havingValue = "redis")
    public CacheManager redisCacheManager(RedisConnectionFactory connectionFactory) {
        RedisCacheConfiguration config = RedisCacheConfiguration.defaultCacheConfig()
                .entryTtl(Duration.ofHours(1))
                .disableCachingNullValues();
        
        return RedisCacheManager.builder(connectionFactory)
                .cacheDefaults(config)
                .build();
    }
    
    @Bean
    @ConditionalOnMissingBean(CacheManager.class)
    public CacheManager simpleCacheManager() {
        SimpleCacheManager cacheManager = new SimpleCacheManager();
        cacheManager.setCaches(Arrays.asList(
            new ConcurrentMapCache("users"),
            new ConcurrentMapCache("products"),
            new ConcurrentMapCache("orders")
        ));
        return cacheManager;
    }
}
```

#### **Service Implementation with Best Practices**
```java
@Service
@Transactional(readOnly = true)
public class OrderService {
    
    private final OrderRepository orderRepository;
    private final ProductService productService;
    private final PaymentService paymentService;
    private final EmailService emailService;
    private final ApplicationEventPublisher eventPublisher;
    
    // Constructor injection (recommended)
    public OrderService(OrderRepository orderRepository,
                       ProductService productService,
                       PaymentService paymentService,
                       EmailService emailService,
                       ApplicationEventPublisher eventPublisher) {
        this.orderRepository = orderRepository;
        this.productService = productService;
        this.paymentService = paymentService;
        this.emailService = emailService;
        this.eventPublisher = eventPublisher;
    }
    
    @Transactional
    public Order createOrder(CreateOrderRequest request) {
        // Validate product availability
        List<Product> products = productService.findAllById(request.getProductIds());
        validateProductAvailability(products, request.getQuantities());
        
        // Create order
        Order order = Order.builder()
                .customerId(request.getCustomerId())
                .products(products)
                .quantities(request.getQuantities())
                .totalAmount(calculateTotal(products, request.getQuantities()))
                .status(OrderStatus.PENDING)
                .build();
        
        Order savedOrder = orderRepository.save(order);
        
        // Process payment
        PaymentResult paymentResult = paymentService.processPayment(
                savedOrder.getTotalAmount(), 
                request.getPaymentMethod()
        );
        
        if (paymentResult.isSuccessful()) {
            savedOrder.setStatus(OrderStatus.PAID);
            savedOrder.setPaymentId(paymentResult.getPaymentId());
            orderRepository.save(savedOrder);
            
            // Publish order created event
            eventPublisher.publishEvent(new OrderCreatedEvent(this, savedOrder));
            
            return savedOrder;
        } else {
            savedOrder.setStatus(OrderStatus.PAYMENT_FAILED);
            orderRepository.save(savedOrder);
            throw new PaymentException("Payment failed: " + paymentResult.getErrorMessage());
        }
    }
    
    @Cacheable(value = "orders", key = "#orderId")
    public Optional<Order> findById(Long orderId) {
        return orderRepository.findById(orderId);
    }
    
    @CacheEvict(value = "orders", key = "#order.id")
    @Transactional
    public Order updateOrderStatus(Order order, OrderStatus newStatus) {
        order.setStatus(newStatus);
        return orderRepository.save(order);
    }
    
    private void validateProductAvailability(List<Product> products, Map<Long, Integer> quantities) {
        for (Product product : products) {
            Integer requestedQuantity = quantities.get(product.getId());
            if (product.getStockQuantity() < requestedQuantity) {
                throw new InsufficientStockException(
                    String.format("Product %s has insufficient stock. Available: %d, Requested: %d",
                            product.getName(), product.getStockQuantity(), requestedQuantity)
                );
            }
        }
    }
    
    private BigDecimal calculateTotal(List<Product> products, Map<Long, Integer> quantities) {
        return products.stream()
                .map(product -> product.getPrice().multiply(BigDecimal.valueOf(quantities.get(product.getId()))))
                .reduce(BigDecimal.ZERO, BigDecimal::add);
    }
}

// Event listener
@Component
public class OrderEventListener {
    
    private final EmailService emailService;
    private final InventoryService inventoryService;
    
    public OrderEventListener(EmailService emailService, InventoryService inventoryService) {
        this.emailService = emailService;
        this.inventoryService = inventoryService;
    }
    
    @EventListener
    @Async
    public void handleOrderCreated(OrderCreatedEvent event) {
        Order order = event.getOrder();
        
        // Send confirmation email
        emailService.sendOrderConfirmation(order);
        
        // Update inventory
        inventoryService.reserveItems(order.getProducts(), order.getQuantities());
        
        log.info("Order {} processed successfully", order.getId());
    }
}
```

### **Testing Spring Beans**

#### **Unit Testing with Mocks**
```java
@ExtendWith(MockitoExtension.class)
class OrderServiceTest {
    
    @Mock
    private OrderRepository orderRepository;
    
    @Mock
    private ProductService productService;
    
    @Mock
    private PaymentService paymentService;
    
    @Mock
    private EmailService emailService;
    
    @Mock
    private ApplicationEventPublisher eventPublisher;
    
    @InjectMocks
    private OrderService orderService;
    
    @Test
    void shouldCreateOrderSuccessfully() {
        // Given
        CreateOrderRequest request = CreateOrderRequest.builder()
                .customerId(1L)
                .productIds(Arrays.asList(1L, 2L))
                .quantities(Map.of(1L, 2, 2L, 1))
                .paymentMethod(PaymentMethod.CREDIT_CARD)
                .build();
        
        List<Product> products = Arrays.asList(
                Product.builder().id(1L).name("Product 1").price(BigDecimal.valueOf(10.00)).stockQuantity(10).build(),
                Product.builder().id(2L).name("Product 2").price(BigDecimal.valueOf(20.00)).stockQuantity(5).build()
        );
        
        when(productService.findAllById(request.getProductIds())).thenReturn(products);
        when(paymentService.processPayment(any(), any()))
                .thenReturn(new PaymentResult(true, "payment-123", null));
        when(orderRepository.save(any(Order.class)))
                .thenAnswer(invocation -> {
                    Order order = invocation.getArgument(0);
                    order.setId(1L);
                    return order;
                });
        
        // When
        Order result = orderService.createOrder(request);
        
        // Then
        assertThat(result.getId()).isEqualTo(1L);
        assertThat(result.getStatus()).isEqualTo(OrderStatus.PAID);
        assertThat(result.getTotalAmount()).isEqualTo(BigDecimal.valueOf(40.00));
        
        verify(eventPublisher).publishEvent(any(OrderCreatedEvent.class));
        verify(orderRepository, times(2)).save(any(Order.class));
    }
    
    @Test
    void shouldThrowExceptionWhenInsufficientStock() {
        // Given
        CreateOrderRequest request = CreateOrderRequest.builder()
                .customerId(1L)
                .productIds(Arrays.asList(1L))
                .quantities(Map.of(1L, 15))  // More than available
                .build();
        
        List<Product> products = Arrays.asList(
                Product.builder().id(1L).name("Product 1").price(BigDecimal.valueOf(10.00)).stockQuantity(5).build()
        );
        
        when(productService.findAllById(request.getProductIds())).thenReturn(products);
        
        // When & Then
        assertThatThrownBy(() -> orderService.createOrder(request))
                .isInstanceOf(InsufficientStockException.class)
                .hasMessageContaining("insufficient stock");
        
        verify(paymentService, never()).processPayment(any(), any());
        verify(orderRepository, never()).save(any(Order.class));
    }
}
```

#### **Integration Testing**
```java
@SpringBootTest
@TestPropertySource(locations = "classpath:application-test.properties")
@Transactional
class OrderServiceIntegrationTest {
    
    @Autowired
    private OrderService orderService;
    
    @Autowired
    private OrderRepository orderRepository;
    
    @Autowired
    private ProductRepository productRepository;
    
    @MockBean
    private PaymentService paymentService;
    
    @MockBean
    private EmailService emailService;
    
    @Test
    void shouldCreateOrderWithRealDatabase() {
        // Given
        Product product1 = productRepository.save(
                Product.builder().name("Test Product 1").price(BigDecimal.valueOf(10.00)).stockQuantity(10).build()
        );
        Product product2 = productRepository.save(
                Product.builder().name("Test Product 2").price(BigDecimal.valueOf(20.00)).stockQuantity(5).build()
        );
        
        CreateOrderRequest request = CreateOrderRequest.builder()
                .customerId(1L)
                .productIds(Arrays.asList(product1.getId(), product2.getId()))
                .quantities(Map.of(product1.getId(), 2, product2.getId(), 1))
                .paymentMethod(PaymentMethod.CREDIT_CARD)
                .build();
        
        when(paymentService.processPayment(any(), any()))
                .thenReturn(new PaymentResult(true, "payment-123", null));
        
        // When
        Order result = orderService.createOrder(request);
        
        // Then
        assertThat(result.getId()).isNotNull();
        
        Optional<Order> savedOrder = orderRepository.findById(result.getId());
        assertThat(savedOrder).isPresent();
        assertThat(savedOrder.get().getStatus()).isEqualTo(OrderStatus.PAID);
        assertThat(savedOrder.get().getTotalAmount()).isEqualTo(BigDecimal.valueOf(40.00));
    }
}
```

---

## 📚 Summary and Best Practices

### **Key Takeaways**

#### **IoC and DI Principles**
- ✅ เข้าใจหลักการ Inversion of Control และประโยชน์
- ✅ ใช้ Constructor Injection เป็นหลัก
- ✅ หลีกเลี่ยง Field Injection ใน production code
- ✅ ใช้ interfaces เพื่อ loose coupling

#### **Bean Management**
- ✅ เข้าใจ Bean Scopes และการใช้งานที่เหมาะสม
- ✅ จัดการ Bean Lifecycle อย่างถูกต้อง
- ✅ ใช้ Conditional annotations สำหรับ flexible configuration
- ✅ เข้าใจ Bean Post-Processors และการใช้งาน

#### **Configuration Best Practices**
- ✅ ใช้ Java Configuration แทน XML
- ✅ แยก configuration ตาม business domain
- ✅ ใช้ @Primary และ @Qualifier อย่างเหมาะสม
- ✅ จัดการ Circular Dependencies อย่างถูกต้อง

### **Common Anti-Patterns to Avoid**

#### **❌ Poor Dependency Injection**
```java
// Avoid field injection in business logic
@Service
public class UserService {
    @Autowired
    private UserRepository userRepository;  // Hard to test
}

// Avoid setter injection for required dependencies
@Service
public class UserService {
    private UserRepository userRepository;
    
    @Autowired
    public void setUserRepository(UserRepository userRepository) {
        this.userRepository = userRepository;  // Can be null
    }
}
```

#### **✅ Good Dependency Injection**
```java
@Service
public class UserService {
    private final UserRepository userRepository;
    
    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;  // Immutable, testable
    }
}
```

#### **❌ Configuration Smells**
```java
// Avoid too many beans in one configuration class
@Configuration
public class MegaConfiguration {
    // 50+ @Bean methods - hard to maintain
}

// Avoid complex conditional logic in @Bean methods
@Bean
public SomeService someService() {
    if (someCondition && anotherCondition && thirdCondition) {
        return new ComplexServiceImpl();
    } else if (fourthCondition) {
        return new AnotherImpl();
    }
    // Too complex
}
```

#### **✅ Good Configuration**
```java
// Modular configuration
@Configuration
public class DatabaseConfiguration {
    // Database-related beans only
}

@Configuration
public class SecurityConfiguration {
    // Security-related beans only
}

// Use conditional annotations instead of complex logic
@Bean
@ConditionalOnProperty(name = "service.type", havingValue = "complex")
public SomeService complexService() {
    return new ComplexServiceImpl();
}

@Bean
@ConditionalOnProperty(name = "service.type", havingValue = "simple")
public SomeService simpleService() {
    return new SimpleServiceImpl();
}
```

---

## 🏋️ Practice Exercises

### **Exercise 1: Dependency Injection Patterns**
1. สร้าง `NotificationService` interface กับ implementations หลายตัว (Email, SMS, Push)
2. ใช้ Constructor injection ใน `UserService`
3. ใช้ `@Qualifier` เพื่อเลือก implementation
4. เขียน unit tests โดยใช้ mocks

### **Exercise 2: Bean Scopes and Lifecycle**
1. สร้าง beans กับ scopes ต่างๆ (singleton, prototype, request)
2. Implement lifecycle methods (`@PostConstruct`, `@PreDestroy`)
3. สร้าง custom `BeanPostProcessor`
4. ทดสอบ lifecycle behavior

### **Exercise 3: Configuration Management**
1. สร้าง modular configuration classes
2. ใช้ `@ConditionalOn...` annotations
3. สร้าง profile-specific configurations
4. Implement custom conditional logic

### **Exercise 4: Advanced DI Patterns**
1. สร้าง custom `FactoryBean`
2. จัดการ circular dependencies
3. Implement method injection patterns
4. สร้าง custom qualifier annotations

---

**🎯 You've mastered Spring Core concepts! Ready for web development?**

**[Next Chapter: Web Development with Spring MVC →](./05-web-development.md)**

---

*Duration: ~4 hours | Difficulty: Intermediate | Prerequisites: Chapter 3 completed*
