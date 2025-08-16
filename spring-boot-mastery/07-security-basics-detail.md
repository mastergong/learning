# Spring Security: การรักษาความปลอดภัยใน Spring Boot Application

## 🎯 จุดประสงค์การเรียนรู้

เมื่อเสร็จสิ้นบทเรียนนี้ คุณจะสามารถ:
- เข้าใจหลักการพื้นฐานของ Spring Security Framework
- กำหนดค่า Authentication และ Authorization อย่างถูกต้อง
- ใช้งาน JWT (JSON Web Tokens) ในการรักษาความปลอดภัย REST APIs
- ประยุกต์ใช้ OAuth2 และ OpenID Connect กับแอปพลิเคชัน Spring Boot
- ออกแบบและสร้างระบบ Role-based Access Control (RBAC)
- จัดการกับ Cross-Site Request Forgery (CSRF) และ Cross-Origin Resource Sharing (CORS)

## 🚀 Spring Security คืออะไร?

Spring Security เป็น framework การรักษาความปลอดภัยที่มีประสิทธิภาพและยืดหยุ่นสำหรับแอปพลิเคชัน Java โดยเฉพาะแอปพลิเคชันที่สร้างบน Spring Framework

```
┌───────────────────────────────────────────────┐
│               Spring Application              │
└───────────────┬───────────────────────────────┘
                │
┌───────────────▼───────────────────────────────┐
│               Spring Security                 │
│                                               │
│  ┌─────────────┐       ┌───────────────────┐  │
│  │ Security    │       │ Access Control    │  │
│  │ Filters     ├──────►│ Decision Manager  │  │
│  └─────────────┘       └─────────┬─────────┘  │
│         │                        │            │
│  ┌──────▼────────┐      ┌────────▼────────┐   │
│  │ Authentication│      │  Authorization  │   │
│  │ Manager       │      │  Manager        │   │
│  └──────┬────────┘      └────────┬────────┘   │
│         │                        │            │
│  ┌──────▼────────┐      ┌────────▼────────┐   │
│  │ Authentication│      │  Security       │   │
│  │ Providers     │      │  Metadata       │   │
│  └──────┬────────┘      └────────┬────────┘   │
│         │                        │            │
│  ┌──────▼────────┐      ┌────────▼────────┐   │
│  │ UserDetails   │      │  Method         │   │
│  │ Service       │      │  Security       │   │
│  └──────┬────────┘      └────────────────┬┘   │
│         │                               │     │
└─────────┼───────────────────────────────┼─────┘
          │                               │
┌─────────▼───────┐             ┌─────────▼─────┐
│  User Storage   │             │  Protected    │
│  (DB, LDAP)     │             │  Resources    │
└─────────────────┘             └───────────────┘
```

### **คุณสมบัติหลักของ Spring Security**

1. **Authentication** - ตรวจสอบว่าผู้ใช้เป็นใคร
2. **Authorization** - ตรวจสอบว่าผู้ใช้มีสิทธิ์เข้าถึงทรัพยากรหรือไม่
3. **Protection** - ป้องกันการโจมตีแบบต่างๆ เช่น CSRF, XSS
4. **Integration** - ทำงานร่วมกับระบบการรักษาความปลอดภัยอื่นๆ เช่น LDAP, OAuth

### **Security Concepts พื้นฐาน**

#### **Principal**
Entity ที่สามารถยืนยันตัวตนได้ในระบบ (เช่น ผู้ใช้, อุปกรณ์, หรือระบบอื่นๆ)

#### **Authentication**
กระบวนการตรวจสอบว่า principal เป็นใคร (ตรวจสอบตัวตน)

#### **Authorization**
กระบวนการตรวจสอบว่า authenticated principal มีสิทธิ์ทำอะไรได้บ้าง

#### **Authority/Role**
สิทธิ์หรือบทบาทที่กำหนดให้กับ principal ว่าสามารถทำอะไรได้บ้าง

## 📋 การเริ่มต้นใช้งาน Spring Security

### **การเพิ่ม Dependency**

**Maven (pom.xml):**
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>

<!-- สำหรับการทดสอบ -->
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-test</artifactId>
    <scope>test</scope>
</dependency>
```

**Gradle (build.gradle):**
```groovy
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-security'
    testImplementation 'org.springframework.security:spring-security-test'
}
```

### **การกำหนดค่า Security ขั้นพื้นฐาน**

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(authz -> authz
                .requestMatchers("/", "/home", "/public/**").permitAll()
                .requestMatchers("/admin/**").hasRole("ADMIN")
                .anyRequest().authenticated()
            )
            .formLogin(form -> form
                .loginPage("/login")
                .permitAll()
            )
            .logout(logout -> logout
                .permitAll()
            );
            
        return http.build();
    }
    
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}
```

### **In-Memory Authentication**

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public UserDetailsService userDetailsService() {
        UserDetails user = User.builder()
                .username("user")
                .password(passwordEncoder().encode("password"))
                .roles("USER")
                .build();
                
        UserDetails admin = User.builder()
                .username("admin")
                .password(passwordEncoder().encode("admin"))
                .roles("USER", "ADMIN")
                .build();
                
        return new InMemoryUserDetailsManager(user, admin);
    }
    
    // ... other beans
}
```

### **Database Authentication**

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Autowired
    private DataSource dataSource;
    
    @Bean
    public UserDetailsService userDetailsService() {
        JdbcUserDetailsManager manager = new JdbcUserDetailsManager(dataSource);
        
        // ถ้าต้องการปรับแต่ง query
        // manager.setUsersByUsernameQuery("select username, password, enabled from users where username=?");
        // manager.setAuthoritiesByUsernameQuery("select username, authority from authorities where username=?");
        
        return manager;
    }
    
    // ... other beans
}
```

### **Custom UserDetailsService**

```java
@Entity
@Table(name = "users")
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(nullable = false, unique = true)
    private String username;
    
    private String password;
    
    private boolean enabled;
    
    @ManyToMany(fetch = FetchType.EAGER)
    @JoinTable(
        name = "user_roles",
        joinColumns = @JoinColumn(name = "user_id"),
        inverseJoinColumns = @JoinColumn(name = "role_id")
    )
    private Set<Role> roles = new HashSet<>();
    
    // Getters and setters
}

@Entity
@Table(name = "roles")
public class Role {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(nullable = false, unique = true)
    private String name;
    
    // Getters and setters
}

@Repository
public interface UserRepository extends JpaRepository<User, Long> {
    Optional<User> findByUsername(String username);
}

@Service
public class CustomUserDetailsService implements UserDetailsService {

    @Autowired
    private UserRepository userRepository;
    
    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        User user = userRepository.findByUsername(username)
                .orElseThrow(() -> new UsernameNotFoundException("User not found: " + username));
        
        Set<GrantedAuthority> authorities = user.getRoles().stream()
                .map(role -> new SimpleGrantedAuthority("ROLE_" + role.getName()))
                .collect(Collectors.toSet());
        
        return new org.springframework.security.core.userdetails.User(
                user.getUsername(),
                user.getPassword(),
                user.isEnabled(),
                true, // account non-expired
                true, // credentials non-expired
                true, // account non-locked
                authorities
        );
    }
}

@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Autowired
    private CustomUserDetailsService userDetailsService;
    
    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
            // ... security configuration
            
        return http.build();
    }
    
    @Bean
    public DaoAuthenticationProvider authenticationProvider() {
        DaoAuthenticationProvider provider = new DaoAuthenticationProvider();
        provider.setUserDetailsService(userDetailsService);
        provider.setPasswordEncoder(passwordEncoder());
        return provider;
    }
    
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}
```

## 🔐 Authentication Methods

### **Form Login**

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(authz -> authz
                // ... authorization rules
            )
            .formLogin(form -> form
                .loginPage("/login")               // Custom login page URL
                .loginProcessingUrl("/authenticate") // URL to submit login data
                .defaultSuccessUrl("/dashboard")    // Redirect after successful login
                .failureUrl("/login?error=true")   // Redirect after failed login
                .usernameParameter("username")     // Parameter name for username
                .passwordParameter("password")     // Parameter name for password
                .permitAll()
            );
            
        return http.build();
    }
    
    // ... other beans
}
```

**Login Form HTML:**
```html
<!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml" xmlns:th="https://www.thymeleaf.org">
    <head>
        <title>Login</title>
    </head>
    <body>
        <div th:if="${param.error}">
            Invalid username or password.
        </div>
        <div th:if="${param.logout}">
            You have been logged out.
        </div>
        <form th:action="@{/authenticate}" method="post">
            <div>
                <label>Username: <input type="text" name="username"/></label>
            </div>
            <div>
                <label>Password: <input type="password" name="password"/></label>
            </div>
            <input type="hidden" th:name="${_csrf.parameterName}" th:value="${_csrf.token}" />
            <div>
                <input type="submit" value="Sign In"/>
            </div>
        </form>
    </body>
</html>
```

### **HTTP Basic Authentication**

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(authz -> authz
                // ... authorization rules
            )
            .httpBasic(Customizer.withDefaults());
            
        return http.build();
    }
    
    // ... other beans
}
```

### **Remember-Me Authentication**

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(authz -> authz
                // ... authorization rules
            )
            .formLogin(form -> form
                // ... form login settings
            )
            .rememberMe(rememberMe -> rememberMe
                .key("uniqueAndSecretKey")          // Key to generate tokens
                .tokenValiditySeconds(86400)         // 1 day
                .rememberMeCookieName("remember-me") // Cookie name
                .rememberMeParameter("remember-me")  // Parameter name
                .tokenRepository(tokenRepository())  // Custom token repository
            );
            
        return http.build();
    }
    
    @Bean
    public PersistentTokenRepository tokenRepository() {
        JdbcTokenRepositoryImpl tokenRepository = new JdbcTokenRepositoryImpl();
        tokenRepository.setDataSource(dataSource);
        return tokenRepository;
    }
    
    // ... other beans
}
```

### **JWT (JSON Web Token) Authentication**

```java
@Configuration
@EnableWebSecurity
public class JwtSecurityConfig {

    @Autowired
    private JwtAuthenticationEntryPoint jwtAuthenticationEntryPoint;
    
    @Autowired
    private CustomUserDetailsService userDetailsService;
    
    @Autowired
    private JwtRequestFilter jwtRequestFilter;
    
    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
            .csrf(csrf -> csrf.disable())
            .authorizeHttpRequests(authz -> authz
                .requestMatchers("/authenticate").permitAll()
                .requestMatchers("/register").permitAll()
                .anyRequest().authenticated()
            )
            .exceptionHandling(ex -> ex.authenticationEntryPoint(jwtAuthenticationEntryPoint))
            .sessionManagement(session -> session.sessionCreationPolicy(SessionCreationPolicy.STATELESS));
        
        // Add JWT filter
        http.addFilterBefore(jwtRequestFilter, UsernamePasswordAuthenticationFilter.class);
        
        return http.build();
    }
    
    @Bean
    public AuthenticationManager authenticationManager(AuthenticationConfiguration authConfig) throws Exception {
        return authConfig.getAuthenticationManager();
    }
    
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}

// JWT Authentication Entry Point
@Component
public class JwtAuthenticationEntryPoint implements AuthenticationEntryPoint {

    @Override
    public void commence(HttpServletRequest request, HttpServletResponse response,
                         AuthenticationException authException) throws IOException {
        response.sendError(HttpServletResponse.SC_UNAUTHORIZED, "Unauthorized");
    }
}

// JWT Utilities
@Component
public class JwtTokenUtil {

    private static final long JWT_TOKEN_VALIDITY = 5 * 60 * 60; // 5 hours
    
    @Value("${jwt.secret}")
    private String secret;
    
    // Generate token for user
    public String generateToken(UserDetails userDetails) {
        Map<String, Object> claims = new HashMap<>();
        return doGenerateToken(claims, userDetails.getUsername());
    }
    
    private String doGenerateToken(Map<String, Object> claims, String subject) {
        return Jwts.builder()
                .setClaims(claims)
                .setSubject(subject)
                .setIssuedAt(new Date(System.currentTimeMillis()))
                .setExpiration(new Date(System.currentTimeMillis() + JWT_TOKEN_VALIDITY * 1000))
                .signWith(SignatureAlgorithm.HS512, secret)
                .compact();
    }
    
    // Validate token
    public Boolean validateToken(String token, UserDetails userDetails) {
        final String username = getUsernameFromToken(token);
        return (username.equals(userDetails.getUsername()) && !isTokenExpired(token));
    }
    
    // Get username from token
    public String getUsernameFromToken(String token) {
        return getClaimFromToken(token, Claims::getSubject);
    }
    
    // Get expiration date from token
    public Date getExpirationDateFromToken(String token) {
        return getClaimFromToken(token, Claims::getExpiration);
    }
    
    public <T> T getClaimFromToken(String token, Function<Claims, T> claimsResolver) {
        final Claims claims = getAllClaimsFromToken(token);
        return claimsResolver.apply(claims);
    }
    
    private Claims getAllClaimsFromToken(String token) {
        return Jwts.parser().setSigningKey(secret).parseClaimsJws(token).getBody();
    }
    
    private Boolean isTokenExpired(String token) {
        final Date expiration = getExpirationDateFromToken(token);
        return expiration.before(new Date());
    }
}

// JWT Request Filter
@Component
public class JwtRequestFilter extends OncePerRequestFilter {

    @Autowired
    private CustomUserDetailsService userDetailsService;
    
    @Autowired
    private JwtTokenUtil jwtTokenUtil;
    
    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain chain)
            throws ServletException, IOException {
        
        final String requestTokenHeader = request.getHeader("Authorization");
        
        String username = null;
        String jwtToken = null;
        
        // JWT Token is in the form "Bearer token". Remove Bearer word and get only the token
        if (requestTokenHeader != null && requestTokenHeader.startsWith("Bearer ")) {
            jwtToken = requestTokenHeader.substring(7);
            try {
                username = jwtTokenUtil.getUsernameFromToken(jwtToken);
            } catch (IllegalArgumentException e) {
                System.out.println("Unable to get JWT Token");
            } catch (ExpiredJwtException e) {
                System.out.println("JWT Token has expired");
            }
        } else {
            logger.warn("JWT Token does not begin with Bearer String");
        }
        
        // Once we get the token validate it
        if (username != null && SecurityContextHolder.getContext().getAuthentication() == null) {
            UserDetails userDetails = this.userDetailsService.loadUserByUsername(username);
            
            // If token is valid, configure Spring Security to manually set authentication
            if (jwtTokenUtil.validateToken(jwtToken, userDetails)) {
                UsernamePasswordAuthenticationToken authenticationToken = 
                        new UsernamePasswordAuthenticationToken(userDetails, null, userDetails.getAuthorities());
                        
                authenticationToken.setDetails(new WebAuthenticationDetailsSource().buildDetails(request));
                
                // After setting the Authentication in the context, we specify that the current user is authenticated
                SecurityContextHolder.getContext().setAuthentication(authenticationToken);
            }
        }
        chain.doFilter(request, response);
    }
}

// Authentication Controller
@RestController
@CrossOrigin
public class JwtAuthenticationController {

    @Autowired
    private AuthenticationManager authenticationManager;
    
    @Autowired
    private JwtTokenUtil jwtTokenUtil;
    
    @Autowired
    private CustomUserDetailsService userDetailsService;
    
    @PostMapping("/authenticate")
    public ResponseEntity<?> createAuthenticationToken(@RequestBody JwtRequest authenticationRequest) throws Exception {
        authenticate(authenticationRequest.getUsername(), authenticationRequest.getPassword());
        
        final UserDetails userDetails = userDetailsService
                .loadUserByUsername(authenticationRequest.getUsername());
                
        final String token = jwtTokenUtil.generateToken(userDetails);
        
        return ResponseEntity.ok(new JwtResponse(token));
    }
    
    private void authenticate(String username, String password) throws Exception {
        try {
            authenticationManager.authenticate(new UsernamePasswordAuthenticationToken(username, password));
        } catch (DisabledException e) {
            throw new Exception("USER_DISABLED", e);
        } catch (BadCredentialsException e) {
            throw new Exception("INVALID_CREDENTIALS", e);
        }
    }
}

// JWT Request Model
public class JwtRequest implements Serializable {
    private String username;
    private String password;
    
    // Default constructor for JSON Parsing
    public JwtRequest() {
    }
    
    public JwtRequest(String username, String password) {
        this.username = username;
        this.password = password;
    }
    
    // Getters and setters
}

// JWT Response Model
public class JwtResponse implements Serializable {
    private String token;
    
    public JwtResponse(String token) {
        this.token = token;
    }
    
    public String getToken() {
        return token;
    }
}
```

### **OAuth2 Authentication**

```java
// OAuth2 Client for Login
@Configuration
@EnableWebSecurity
public class OAuth2SecurityConfig {

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(authz -> authz
                .requestMatchers("/", "/login**", "/error**").permitAll()
                .anyRequest().authenticated()
            )
            .oauth2Login(oauth2 -> oauth2
                .loginPage("/login")
                .defaultSuccessUrl("/dashboard")
                .failureUrl("/login?error=true")
                .userInfoEndpoint(userInfo -> userInfo
                    .userService(oauth2UserService())
                )
            );
            
        return http.build();
    }
    
    @Bean
    public OAuth2UserService<OAuth2UserRequest, OAuth2User> oauth2UserService() {
        DefaultOAuth2UserService delegate = new DefaultOAuth2UserService();
        
        return (userRequest) -> {
            OAuth2User oauth2User = delegate.loadUser(userRequest);
            
            // Extract client registration ID (e.g., "google", "facebook")
            String registrationId = userRequest.getClientRegistration().getRegistrationId();
            
            // Extract OAuth2 attributes based on provider
            Map<String, Object> attributes = oauth2User.getAttributes();
            String email = null;
            String name = null;
            
            if ("google".equals(registrationId)) {
                email = (String) attributes.get("email");
                name = (String) attributes.get("name");
            } else if ("facebook".equals(registrationId)) {
                email = (String) attributes.get("email");
                name = (String) attributes.get("name");
            } else if ("github".equals(registrationId)) {
                email = (String) attributes.get("email");
                name = (String) attributes.get("name");
            }
            
            // Here you can process the user information:
            // - Create new user if it doesn't exist
            // - Update existing user information
            // - Assign roles based on the OAuth2 provider or user attributes
            
            return oauth2User;
        };
    }
}

// In application.yml
spring:
  security:
    oauth2:
      client:
        registration:
          google:
            client-id: your-google-client-id
            client-secret: your-google-client-secret
            scope:
              - email
              - profile
          facebook:
            client-id: your-facebook-client-id
            client-secret: your-facebook-client-secret
            scope:
              - email
              - public_profile
          github:
            client-id: your-github-client-id
            client-secret: your-github-client-secret
            scope:
              - user:email
              - read:user
```

### **Multi-Factor Authentication (MFA)**

```java
@Configuration
@EnableWebSecurity
public class MfaSecurityConfig {

    @Autowired
    private CustomUserDetailsService userDetailsService;
    
    @Autowired
    private MfaAuthenticationProvider mfaAuthenticationProvider;
    
    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(authz -> authz
                .requestMatchers("/login", "/mfa/**", "/register").permitAll()
                .anyRequest().authenticated()
            )
            .formLogin(form -> form
                .loginPage("/login")
                .permitAll()
            );
            
        return http.build();
    }
    
    @Bean
    public AuthenticationManager authenticationManager(AuthenticationConfiguration authConfig) throws Exception {
        return authConfig.getAuthenticationManager();
    }
    
    @Bean
    public DaoAuthenticationProvider daoAuthenticationProvider() {
        DaoAuthenticationProvider provider = new DaoAuthenticationProvider();
        provider.setUserDetailsService(userDetailsService);
        provider.setPasswordEncoder(passwordEncoder());
        return provider;
    }
    
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
    
    @Autowired
    public void configureGlobal(AuthenticationManagerBuilder auth) throws Exception {
        auth.authenticationProvider(daoAuthenticationProvider());
        auth.authenticationProvider(mfaAuthenticationProvider);
    }
}

// MFA Authentication Provider
@Component
public class MfaAuthenticationProvider implements AuthenticationProvider {

    @Autowired
    private MfaTokenService mfaTokenService;
    
    @Override
    public Authentication authenticate(Authentication authentication) throws AuthenticationException {
        MfaAuthenticationToken mfaToken = (MfaAuthenticationToken) authentication;
        
        // Get the pre-authenticated token from the first step of authentication
        String username = mfaToken.getPrincipal().toString();
        String verificationCode = mfaToken.getCredentials().toString();
        
        // Validate the MFA code
        if (mfaTokenService.validateToken(username, verificationCode)) {
            // Load user authorities/roles
            Set<GrantedAuthority> authorities = mfaTokenService.getUserAuthorities(username);
            
            // Create fully authenticated token
            MfaAuthenticationToken authenticatedToken = new MfaAuthenticationToken(
                    username, verificationCode, authorities);
            
            return authenticatedToken;
        }
        
        throw new BadCredentialsException("Invalid verification code");
    }
    
    @Override
    public boolean supports(Class<?> authentication) {
        return MfaAuthenticationToken.class.isAssignableFrom(authentication);
    }
}

// MFA Authentication Token
public class MfaAuthenticationToken extends AbstractAuthenticationToken {

    private final Object principal;
    private Object credentials;
    
    public MfaAuthenticationToken(Object principal, Object credentials) {
        super(null);
        this.principal = principal;
        this.credentials = credentials;
        setAuthenticated(false);
    }
    
    public MfaAuthenticationToken(Object principal, Object credentials, 
            Collection<? extends GrantedAuthority> authorities) {
        super(authorities);
        this.principal = principal;
        this.credentials = credentials;
        super.setAuthenticated(true);
    }
    
    @Override
    public Object getCredentials() {
        return this.credentials;
    }
    
    @Override
    public Object getPrincipal() {
        return this.principal;
    }
    
    @Override
    public void setAuthenticated(boolean isAuthenticated) throws IllegalArgumentException {
        if (isAuthenticated) {
            throw new IllegalArgumentException(
                "Cannot set this token to trusted - use constructor which takes a GrantedAuthority list instead");
        }
        super.setAuthenticated(false);
    }
    
    @Override
    public void eraseCredentials() {
        super.eraseCredentials();
        credentials = null;
    }
}

// MFA Token Service
@Service
public class MfaTokenService {

    @Autowired
    private UserRepository userRepository;
    
    // Generate a random 6-digit code as MFA token
    public String generateToken(String username) {
        SecureRandom random = new SecureRandom();
        int code = 100000 + random.nextInt(900000); // 6-digit code
        
        // Store the code with username and timestamp
        // This could be in a database, Redis, or other storage
        
        return String.valueOf(code);
    }
    
    public boolean validateToken(String username, String token) {
        // Retrieve stored token for the user
        // Check if token matches and is not expired
        
        // For demo purposes, let's assume it's valid
        return true;
    }
    
    public Set<GrantedAuthority> getUserAuthorities(String username) {
        // Retrieve user roles from database
        User user = userRepository.findByUsername(username)
                .orElseThrow(() -> new UsernameNotFoundException("User not found: " + username));
        
        Set<GrantedAuthority> authorities = user.getRoles().stream()
                .map(role -> new SimpleGrantedAuthority("ROLE_" + role.getName()))
                .collect(Collectors.toSet());
                
        return authorities;
    }
}

// MFA Authentication Controller
@Controller
@RequestMapping("/mfa")
public class MfaController {

    @Autowired
    private MfaTokenService mfaTokenService;
    
    @Autowired
    private AuthenticationManager authenticationManager;
    
    @GetMapping("/verify")
    public String getVerificationPage(Model model) {
        return "mfa-verification";
    }
    
    @PostMapping("/verify")
    public String verifyMfaCode(
            @RequestParam("code") String code,
            @RequestParam("username") String username,
            HttpSession session) {
        
        try {
            // Create MFA authentication token
            MfaAuthenticationToken token = new MfaAuthenticationToken(username, code);
            
            // Authenticate with MFA
            Authentication authentication = authenticationManager.authenticate(token);
            
            // Set authentication in security context
            SecurityContextHolder.getContext().setAuthentication(authentication);
            
            // Redirect to secured page
            return "redirect:/dashboard";
        } catch (AuthenticationException e) {
            // Failed verification
            return "redirect:/mfa/verify?error=true";
        }
    }
    
    @GetMapping("/send")
    @ResponseBody
    public String sendVerificationCode(@RequestParam("username") String username) {
        // Generate and send verification code
        String code = mfaTokenService.generateToken(username);
        
        // In a real application, send this code via email, SMS, or other channel
        // For demo, we'll just return it
        return "Verification code sent: " + code;
    }
}
```

## 🔒 Authorization

### **URL-Based Authorization**

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(authz -> authz
                // Public URLs
                .requestMatchers("/", "/home", "/register", "/css/**", "/js/**").permitAll()
                
                // Admin URLs
                .requestMatchers("/admin/**").hasRole("ADMIN")
                
                // User profile URLs
                .requestMatchers("/user/profile/**").hasAnyRole("USER", "ADMIN")
                
                // API URLs with different roles
                .requestMatchers(HttpMethod.GET, "/api/products").permitAll()
                .requestMatchers(HttpMethod.POST, "/api/products").hasRole("EDITOR")
                .requestMatchers(HttpMethod.PUT, "/api/products").hasRole("EDITOR")
                .requestMatchers(HttpMethod.DELETE, "/api/products").hasRole("ADMIN")
                
                // Match with patterns
                .requestMatchers("/reports/*/view").hasRole("USER")
                .requestMatchers("/reports/*/edit").hasRole("EDITOR")
                
                // Any other request
                .anyRequest().authenticated()
            );
            
        return http.build();
    }
    
    // ... other beans
}
```

### **Method-Level Security**

```java
@Configuration
@EnableMethodSecurity
public class MethodSecurityConfig {
    // Method security is enabled by the annotation
}

@Service
public class UserService {

    // Only users with ADMIN role can access
    @PreAuthorize("hasRole('ADMIN')")
    public List<User> getAllUsers() {
        // ...
    }
    
    // Access based on SpEL expressions
    @PreAuthorize("hasRole('ADMIN') or hasRole('MANAGER')")
    public User getUserById(Long id) {
        // ...
    }
    
    // Access only if username matches or user is admin
    @PreAuthorize("#username == authentication.principal.username or hasRole('ADMIN')")
    public UserDetails getUserDetailsByUsername(String username) {
        // ...
    }
    
    // Secure method based on return value
    @PostAuthorize("returnObject.username == authentication.principal.username")
    public User loadUserByEmail(String email) {
        // ...
    }
    
    // Secure arguments with @PreFilter
    @PreFilter("filterObject.username != authentication.principal.username")
    public List<User> deleteUsers(List<User> users) {
        // ...
    }
    
    // Secure return value with @PostFilter
    @PostFilter("filterObject.owner == authentication.principal.username")
    public List<Document> getDocuments() {
        // ...
    }
    
    // Secure with custom security expression
    @PreAuthorize("@userSecurityService.checkUserId(#id)")
    public User updateUser(Long id, UserDto userDto) {
        // ...
    }
}

// Custom security expression bean
@Component
public class UserSecurityService {

    @Autowired
    private UserRepository userRepository;
    
    public boolean checkUserId(Long id) {
        // Get currently authenticated user
        Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
        String currentUsername = authentication.getName();
        
        // Check if user is authorized to access this user ID
        User user = userRepository.findById(id).orElse(null);
        if (user == null) {
            return false;
        }
        
        // Either the user is accessing their own account or they're an admin
        return user.getUsername().equals(currentUsername) ||
                authentication.getAuthorities().stream()
                        .anyMatch(a -> a.getAuthority().equals("ROLE_ADMIN"));
    }
}
```

### **Domain Object Security**

```java
@Configuration
@EnableMethodSecurity
public class SecurityConfig {

    @Bean
    public SecurityExpressionHandler<FilterInvocation> webExpressionHandler() {
        DefaultWebSecurityExpressionHandler handler = new DefaultWebSecurityExpressionHandler();
        handler.setPermissionEvaluator(permissionEvaluator());
        return handler;
    }
    
    @Bean
    public PermissionEvaluator permissionEvaluator() {
        return new DomainObjectPermissionEvaluator();
    }
}

// Custom Permission Evaluator
public class DomainObjectPermissionEvaluator implements PermissionEvaluator {

    @Autowired
    private DocumentRepository documentRepository;
    
    @Override
    public boolean hasPermission(Authentication authentication, Object targetDomainObject, Object permission) {
        if (authentication == null || targetDomainObject == null || !(permission instanceof String)) {
            return false;
        }
        
        String permissionString = (String) permission;
        
        if (targetDomainObject instanceof Document) {
            Document document = (Document) targetDomainObject;
            return hasDocumentPermission(authentication, document, permissionString);
        }
        
        return false;
    }
    
    @Override
    public boolean hasPermission(Authentication authentication, Serializable targetId, String targetType, Object permission) {
        if (authentication == null || targetId == null || targetType == null || !(permission instanceof String)) {
            return false;
        }
        
        String permissionString = (String) permission;
        
        if ("document".equals(targetType)) {
            Optional<Document> document = documentRepository.findById((Long) targetId);
            return document.isPresent() && hasDocumentPermission(authentication, document.get(), permissionString);
        }
        
        return false;
    }
    
    private boolean hasDocumentPermission(Authentication authentication, Document document, String permission) {
        String username = authentication.getName();
        
        // Check document ownership
        if (document.getOwner().equals(username)) {
            // Owner can do anything
            return true;
        }
        
        // Check shared permissions
        if (document.getSharedWith().contains(username)) {
            // Shared users can read and comment but not delete
            return "read".equals(permission) || "comment".equals(permission);
        }
        
        // Check admin role
        return authentication.getAuthorities().stream()
                .anyMatch(a -> a.getAuthority().equals("ROLE_ADMIN"));
    }
}

// Using in service methods
@Service
public class DocumentService {

    // Using domain object security
    @PreAuthorize("hasPermission(#document, 'write')")
    public Document updateDocument(Document document) {
        // ...
    }
    
    @PreAuthorize("hasPermission(#id, 'document', 'read')")
    public Document getDocument(Long id) {
        // ...
    }
}
```

## 🌐 CORS และ CSRF Protection

### **CORS Configuration**

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
            // ... other security configuration
            .cors(cors -> cors.configurationSource(corsConfigurationSource()));
            
        return http.build();
    }
    
    @Bean
    public CorsConfigurationSource corsConfigurationSource() {
        CorsConfiguration configuration = new CorsConfiguration();
        configuration.setAllowedOrigins(Arrays.asList("https://example.com", "https://api.example.com"));
        configuration.setAllowedMethods(Arrays.asList("GET", "POST", "PUT", "PATCH", "DELETE", "OPTIONS"));
        configuration.setAllowedHeaders(Arrays.asList("authorization", "content-type", "x-auth-token"));
        configuration.setExposedHeaders(Arrays.asList("x-auth-token"));
        configuration.setAllowCredentials(true);
        configuration.setMaxAge(3600L);
        
        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        source.registerCorsConfiguration("/**", configuration);
        return source;
    }
}

// Alternative: Using Spring MVC to configure CORS
@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/api/**")
            .allowedOrigins("https://example.com")
            .allowedMethods("GET", "POST", "PUT", "DELETE")
            .allowedHeaders("*")
            .allowCredentials(true)
            .maxAge(3600);
    }
}

// Controller-level CORS
@RestController
@RequestMapping("/api/users")
@CrossOrigin(origins = "https://example.com", maxAge = 3600)
public class UserController {
    // ...
}

// Method-level CORS
@RestController
@RequestMapping("/api/documents")
public class DocumentController {

    @GetMapping
    @CrossOrigin(origins = "https://example.com")
    public List<Document> getAllDocuments() {
        // ...
    }
    
    @PostMapping
    @CrossOrigin(origins = {"https://admin.example.com", "https://editor.example.com"})
    public Document createDocument(@RequestBody DocumentDto documentDto) {
        // ...
    }
}
```

### **CSRF Protection**

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
            // ... other security configuration
            .csrf(csrf -> csrf
                .csrfTokenRepository(CookieCsrfTokenRepository.withHttpOnlyFalse())
                .ignoringRequestMatchers("/api/webhook/**") // Disable for specific endpoints
            );
            
        return http.build();
    }
}

// Custom CSRF token repository
@Component
public class CustomCsrfTokenRepository implements CsrfTokenRepository {

    @Override
    public CsrfToken generateToken(HttpServletRequest request) {
        return new DefaultCsrfToken("X-CSRF-TOKEN", "_csrf", 
                UUID.randomUUID().toString());
    }
    
    @Override
    public void saveToken(CsrfToken token, HttpServletRequest request, HttpServletResponse response) {
        if (token == null) {
            response.setHeader("X-CSRF-TOKEN", "");
            return;
        }
        response.setHeader("X-CSRF-TOKEN", token.getToken());
    }
    
    @Override
    public CsrfToken loadToken(HttpServletRequest request) {
        String token = request.getHeader("X-CSRF-TOKEN");
        if (token == null) {
            return null;
        }
        return new DefaultCsrfToken("X-CSRF-TOKEN", "_csrf", token);
    }
}

// Disable CSRF for REST APIs
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
            .securityMatcher("/api/**")
            .csrf(csrf -> csrf.disable())
            .authorizeHttpRequests(authz -> authz.anyRequest().authenticated())
            .sessionManagement(session -> session.sessionCreationPolicy(SessionCreationPolicy.STATELESS));
            
        return http.build();
    }
}
```

## 🛡️ Security Headers and Protection against Common Attacks

### **Security Headers Configuration**

```java
@Configuration
@EnableWebSecurity
public class SecurityHeadersConfig {

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
            // ... other security configurations
            .headers(headers -> headers
                // Default protection
                .defaultsDisabled()
                
                // Cache Control
                .cacheControl(cache -> {})
                
                // Content Type Options
                .contentTypeOptions(content -> {})
                
                // Frame Options (prevent clickjacking)
                .frameOptions(frame -> frame.deny())
                
                // HTTP Strict Transport Security
                .httpStrictTransportSecurity(hsts -> hsts
                    .includeSubDomains(true)
                    .maxAgeInSeconds(31536000)
                    .preload(true))
                
                // X-XSS-Protection
                .xssProtection(xss -> xss.block(true))
                
                // Content Security Policy
                .contentSecurityPolicy(csp -> 
                    csp.policyDirectives("default-src 'self'; script-src 'self' https://trusted.cdn.com"))
                
                // Referrer Policy
                .referrerPolicy(referrer -> 
                    referrer.policy(ReferrerPolicyHeaderWriter.ReferrerPolicy.SAME_ORIGIN))
                
                // Feature Policy
                .permissionsPolicy(permissions -> permissions
                    .policy("camera=(), microphone=(), geolocation=(self)"))
            );
            
        return http.build();
    }
}
```

### **Protection against Common Attacks**

```java
@Configuration
@EnableWebSecurity
public class SecurityAttacksProtectionConfig {

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
            // Prevent brute force attacks
            .sessionManagement(session -> session
                .maximumSessions(1)
                .maxSessionsPreventsLogin(true))
                
            // Prevent session fixation
            .sessionManagement(session -> session
                .sessionFixation().changeSessionId())
                
            // HTTPS redirect
            .requiresChannel(channel -> channel
                .anyRequest().requiresSecure())
                
            // Login security
            .formLogin(form -> form
                .failureHandler(failureHandler())
                .successHandler(successHandler()));
            
        return http.build();
    }
    
    @Bean
    public AuthenticationFailureHandler failureHandler() {
        return new CustomAuthenticationFailureHandler();
    }
    
    @Bean
    public AuthenticationSuccessHandler successHandler() {
        return new CustomAuthenticationSuccessHandler();
    }
}

// Custom authentication handlers to prevent brute force attacks
@Component
public class CustomAuthenticationFailureHandler implements AuthenticationFailureHandler {

    private static final int MAX_ATTEMPTS = 5;
    private final Map<String, LoginAttempt> attemptsCache = new ConcurrentHashMap<>();
    
    @Override
    public void onAuthenticationFailure(HttpServletRequest request, HttpServletResponse response,
            AuthenticationException exception) throws IOException, ServletException {
        
        String username = request.getParameter("username");
        LoginAttempt attempt = attemptsCache.getOrDefault(username, new LoginAttempt());
        attempt.increment();
        attemptsCache.put(username, attempt);
        
        if (attempt.getCount() >= MAX_ATTEMPTS) {
            // Handle locked account - redirect or show error
            response.sendRedirect("/login?locked=true");
        } else {
            // Normal failure redirect
            response.sendRedirect("/login?error=true");
        }
    }
}

@Component
public class CustomAuthenticationSuccessHandler implements AuthenticationSuccessHandler {

    private final Map<String, LoginAttempt> attemptsCache;
    
    @Autowired
    public CustomAuthenticationSuccessHandler(CustomAuthenticationFailureHandler failureHandler) {
        // Inject the same cache used by the failure handler
        this.attemptsCache = failureHandler.getAttemptsCache();
    }
    
    @Override
    public void onAuthenticationSuccess(HttpServletRequest request, HttpServletResponse response,
            Authentication authentication) throws IOException, ServletException {
        
        String username = authentication.getName();
        
        // Reset login attempts on successful login
        attemptsCache.remove(username);
        
        // Redirect to appropriate page based on roles
        Collection<? extends GrantedAuthority> authorities = authentication.getAuthorities();
        boolean isAdmin = authorities.stream()
                .anyMatch(a -> a.getAuthority().equals("ROLE_ADMIN"));
        
        if (isAdmin) {
            response.sendRedirect("/admin/dashboard");
        } else {
            response.sendRedirect("/dashboard");
        }
    }
}

class LoginAttempt {
    private int count = 0;
    private LocalDateTime lastAttempt = LocalDateTime.now();
    
    public void increment() {
        count++;
        lastAttempt = LocalDateTime.now();
    }
    
    public int getCount() {
        // Reset count if last attempt was more than 1 hour ago
        if (lastAttempt.plusHours(1).isBefore(LocalDateTime.now())) {
            count = 0;
        }
        return count;
    }
}
```

## 🧪 แบบฝึกหัด

### **แบบฝึกหัดที่ 1: การตั้งค่า Basic Authentication**

สร้างแอปพลิเคชัน Spring Boot ที่มี:
1. Basic Authentication
2. การเก็บข้อมูลผู้ใช้ใน database
3. แยก access control ระหว่าง USER และ ADMIN roles
4. เข้าถึง public API endpoints โดยไม่ต้อง authenticate

### **แบบฝึกหัดที่ 2: การสร้างระบบ JWT Authentication**

สร้าง RESTful API ที่มี:
1. การลงทะเบียนผู้ใช้ใหม่
2. การ login และได้รับ JWT token
3. การเข้าถึง protected endpoints ด้วย JWT
4. การต่ออายุ token และการ logout

### **แบบฝึกหัดที่ 3: การใช้ Method Security**

ปรับปรุงระบบจากแบบฝึกหัดที่ 2 โดย:
1. เพิ่ม method-level security ด้วย @PreAuthorize
2. ใช้ SpEL expressions เพื่อจำกัดการเข้าถึงตามเงื่อนไขต่างๆ
3. สร้าง custom security expressions
4. ใช้ @PostAuthorize เพื่อตรวจสอบข้อมูลที่ส่งกลับ

### **แบบฝึกหัดที่ 4: การสร้างระบบ OAuth2**

ปรับปรุงแอปพลิเคชันให้รองรับ:
1. การ login ด้วย Google หรือ Facebook
2. การเชื่อมโยงบัญชี social กับบัญชีในระบบ
3. การกำหนดสิทธิ์ตาม role จากข้อมูลที่ได้จาก OAuth provider

## 📝 สรุป

Spring Security เป็น framework ที่ทรงพลังสำหรับการรักษาความปลอดภัยในแอปพลิเคชัน Spring Boot โดยให้คุณสมบัติต่างๆ มากมาย:

### **Authentication**
- Form-based Authentication
- HTTP Basic/Digest Authentication
- JWT Token Authentication
- OAuth2/OpenID Connect
- Remember-me Authentication
- Multi-factor Authentication

### **Authorization**
- URL-based Authorization
- Method Security
- Role-based Access Control
- Expression-based Access Control
- Domain Object Security

### **Protection against Attacks**
- CSRF Protection
- CORS Configuration
- Security Headers
- Session Management
- Brute Force Protection

การเลือกใช้งาน Spring Security ช่วยให้คุณสามารถสร้างแอปพลิเคชันที่มีการรักษาความปลอดภัยระดับสูงได้อย่างรวดเร็วและมีประสิทธิภาพ โดยไม่ต้องเขียนโค้ดเพื่อจัดการระบบความปลอดภัยเองทั้งหมด

## 🔍 อ้างอิงเพิ่มเติม

- [Spring Security Official Documentation](https://docs.spring.io/spring-security/reference/index.html)
- [Spring Security Architecture Guide](https://spring.io/guides/topicals/spring-security-architecture)
- [OAuth2 with Spring Boot](https://spring.io/guides/tutorials/spring-boot-oauth2/)
- [JWT Authentication with Spring Security](https://auth0.com/blog/implementing-jwt-authentication-on-spring-boot/)
- [OWASP Top Ten Security Risks](https://owasp.org/www-project-top-ten/)

---

**[⬅️ กลับไปที่บทที่ 6: Data Access](./06-data-access.md) | [➡️ ไปที่บทที่ 8: Testing Fundamentals](./08-testing-fundamentals.md)**
