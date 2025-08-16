# Chapter 2: Development Environment Setup
## การติดตั้งและจัดเตรียม Environment สำหรับการพัฒนา Spring Boot

### 🎯 Learning Objectives
หลังจากเรียนบทนี้แล้ว คุณจะสามารถ:
- ติดตั้งและ configure IntelliJ IDEA สำหรับ Spring Boot development
- เข้าใจและใช้งาน Maven และ Gradle build tools
- ตั้งค่า Git version control และ best practices
- ใช้งาน Docker สำหรับ containerization
- สร้าง development workflow ที่มีประสิทธิภาพ

---

## ☕ Java Development Kit (JDK) Setup

### **JDK Version Selection**

#### **Recommended Java Versions**
```bash
# Java 17 (LTS) - Recommended for Spring Boot 3.x
Java 17.0.8+7-LTS

# Java 21 (Latest LTS) - For cutting-edge features
Java 21.0.1+12-LTS

# Java 11 (Legacy LTS) - For Spring Boot 2.x projects
Java 11.0.20+8-LTS
```

#### **JDK Installation - Windows**
```powershell
# Option 1: Using Chocolatey
choco install openjdk17

# Option 2: Using Scoop
scoop install openjdk17

# Option 3: Manual installation
# 1. Download from https://adoptium.net/
# 2. Run installer
# 3. Set JAVA_HOME environment variable

# Verify installation
java -version
javac -version
echo $env:JAVA_HOME
```

#### **JDK Installation - macOS**
```bash
# Option 1: Using Homebrew
brew install openjdk@17

# Option 2: Using SDKMAN
curl -s "https://get.sdkman.io" | bash
sdk install java 17.0.8-tem

# Verify installation
java -version
javac -version
echo $JAVA_HOME
```

#### **JDK Installation - Linux**
```bash
# Ubuntu/Debian
sudo apt update
sudo apt install openjdk-17-jdk

# CentOS/RHEL
sudo yum install java-17-openjdk-devel

# Using SDKMAN (recommended)
curl -s "https://get.sdkman.io" | bash
sdk install java 17.0.8-tem

# Verify installation
java -version
javac -version
echo $JAVA_HOME
```

### **Managing Multiple JDK Versions**

#### **Using SDKMAN (Cross-platform)**
```bash
# Install SDKMAN
curl -s "https://get.sdkman.io" | bash
source "$HOME/.sdkman/bin/sdkman-init.sh"

# List available Java versions
sdk list java

# Install multiple versions
sdk install java 17.0.8-tem
sdk install java 21.0.1-tem
sdk install java 11.0.20-tem

# Switch between versions
sdk use java 17.0.8-tem
sdk default java 17.0.8-tem

# Current version
sdk current java
```

#### **Using jenv (macOS/Linux)**
```bash
# Install jenv
brew install jenv  # macOS
# or
git clone https://github.com/jenv/jenv.git ~/.jenv  # Linux

# Add to shell profile
echo 'export PATH="$HOME/.jenv/bin:$PATH"' >> ~/.zshrc
echo 'eval "$(jenv init -)"' >> ~/.zshrc

# Add JDK versions
jenv add /usr/lib/jvm/java-17-openjdk-amd64
jenv add /usr/lib/jvm/java-21-openjdk-amd64

# List versions
jenv versions

# Set global version
jenv global 17.0.8
```

---

## 💻 IntelliJ IDEA Setup

### **Installation and Basic Configuration**

#### **IntelliJ IDEA Editions**
- **Community Edition** - Free, sufficient for Spring Boot development
- **Ultimate Edition** - Paid, advanced features for enterprise development

#### **Installation Process**
```bash
# Windows - Using Chocolatey
choco install intellijidea-community
# or
choco install intellijidea-ultimate

# macOS - Using Homebrew Cask
brew install --cask intellij-idea-ce
# or
brew install --cask intellij-idea

# Linux - Using Snap
sudo snap install intellij-idea-community --classic
# or
sudo snap install intellij-idea-ultimate --classic
```

### **Essential IntelliJ IDEA Plugins for Spring Boot**

#### **Core Spring Plugins**
```bash
# Essential plugins to install:
1. Spring Boot                    # Enhanced Spring Boot support
2. Spring MVC                     # Spring MVC framework support
3. Spring Data                    # Spring Data integration
4. Spring Security                # Security framework support
5. Lombok                         # Lombok annotations support
6. Database Tools and SQL         # Database integration
7. Docker                         # Docker integration
8. GitToolBox                     # Enhanced Git features
9. SonarLint                      # Code quality analysis
10. Maven Helper                  # Maven dependency analysis
```

#### **Plugin Installation**
```bash
# File → Settings → Plugins → Marketplace
# Search and install each plugin
# Restart IDE when prompted
```

### **IntelliJ IDEA Configuration for Spring Boot**

#### **Project Settings**
```bash
# File → Settings → Build, Execution, Deployment → Build Tools → Maven
✅ Import Maven projects automatically
✅ Generate sources and documentation
✅ Automatically download sources
✅ Automatically download documentation

# File → Settings → Build, Execution, Deployment → Compiler → Annotation Processors
✅ Enable annotation processing
✅ Obtain processors from project classpath
```

#### **Code Style Configuration**
```bash
# File → Settings → Editor → Code Style → Java
# Import Spring Boot code style (optional)
# Or configure manually:

Indentation:
- Use tab character: ❌
- Tab size: 4
- Indent: 4
- Continuation indent: 8

Spaces:
✅ Before method parentheses: if declaration
✅ Around operators: =, +, -, etc.
✅ Before left brace: class, method, if, etc.

Wrapping:
- Method parameters: Chop down if long
- Method call arguments: Chop down if long
- Chained method calls: Chop down if long
```

#### **Live Templates for Spring Boot**
```java
// Custom live templates (File → Settings → Editor → Live Templates)

// @Service class template
@abbreviation("service")
@Service
public class $CLASS_NAME$ {
    private static final Logger log = LoggerFactory.getLogger($CLASS_NAME$.class);
    
    $END$
}

// @RestController template  
@abbreviation("controller")
@RestController
@RequestMapping("/api/$PATH$")
public class $CLASS_NAME$ {
    private static final Logger log = LoggerFactory.getLogger($CLASS_NAME$.class);
    
    $END$
}

// @Entity template
@abbreviation("entity")
@Entity
@Table(name = "$TABLE_NAME$")
public class $CLASS_NAME$ {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    $END$
}
```

### **IntelliJ IDEA Shortcuts for Spring Boot Development**

#### **Essential Shortcuts**
```bash
# Navigation
Ctrl+N             # Go to class
Ctrl+Shift+N       # Go to file
Ctrl+Alt+Shift+N   # Go to symbol
Ctrl+B             # Go to declaration
Ctrl+Alt+B         # Go to implementations

# Code Generation
Alt+Insert         # Generate code (Constructor, Getter/Setter, etc.)
Ctrl+O             # Override methods
Ctrl+I             # Implement methods

# Refactoring
Shift+F6           # Rename
Ctrl+Alt+M         # Extract method
Ctrl+Alt+V         # Extract variable
Ctrl+Alt+C         # Extract constant

# Running and Debugging
Shift+F10          # Run
Shift+F9           # Debug
Ctrl+F2            # Stop
Ctrl+Shift+F10     # Run context

# Spring Boot Specific
Ctrl+Shift+A       # Find action
Ctrl+Alt+S         # Settings
Ctrl+`             # Switch between tools
```

---

## 📦 Build Tools: Maven & Gradle

### **Apache Maven Setup**

#### **Maven Installation**
```bash
# Windows - Using Chocolatey
choco install maven

# macOS - Using Homebrew
brew install maven

# Linux - Package manager
sudo apt install maven  # Ubuntu/Debian
sudo yum install maven  # CentOS/RHEL

# Manual installation
# 1. Download from https://maven.apache.org/download.cgi
# 2. Extract to /opt/maven (Linux/Mac) or C:\maven (Windows)
# 3. Add to PATH environment variable
# 4. Set M2_HOME environment variable

# Verify installation
mvn -version
```

#### **Maven Configuration**

**Global Settings (`~/.m2/settings.xml`):**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0">
    
    <!-- Local repository location -->
    <localRepository>${user.home}/.m2/repository</localRepository>
    
    <!-- Proxy settings (if needed) -->
    <proxies>
        <proxy>
            <id>corporate-proxy</id>
            <active>false</active>
            <protocol>http</protocol>
            <host>proxy.company.com</host>
            <port>8080</port>
            <nonProxyHosts>localhost|127.0.0.1</nonProxyHosts>
        </proxy>
    </proxies>
    
    <!-- Maven Central mirror (optional) -->
    <mirrors>
        <mirror>
            <id>maven-central-mirror</id>
            <mirrorOf>central</mirrorOf>
            <name>Maven Central Mirror</name>
            <url>https://repo1.maven.org/maven2</url>
        </mirror>
    </mirrors>
    
    <!-- Default profiles -->
    <profiles>
        <profile>
            <id>development</id>
            <activation>
                <activeByDefault>true</activeByDefault>
            </activation>
            <properties>
                <maven.compiler.source>17</maven.compiler.source>
                <maven.compiler.target>17</maven.compiler.target>
                <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
            </properties>
        </profile>
    </profiles>
</settings>
```

#### **Essential Maven Commands**
```bash
# Project lifecycle
mvn clean                    # Clean target directory
mvn compile                  # Compile source code
mvn test                     # Run tests
mvn package                  # Create JAR/WAR
mvn install                  # Install to local repository
mvn deploy                   # Deploy to remote repository

# Spring Boot specific
mvn spring-boot:run          # Run Spring Boot application
mvn spring-boot:build-image  # Build Docker image

# Dependency management
mvn dependency:tree          # Show dependency tree
mvn dependency:analyze       # Analyze dependencies
mvn dependency:resolve       # Download dependencies

# Useful combinations
mvn clean compile           # Clean and compile
mvn clean test             # Clean and test
mvn clean package          # Clean and package
mvn clean install          # Full build and install

# Skip tests (use sparingly)
mvn clean package -DskipTests
mvn clean install -Dmaven.test.skip=true
```

### **Gradle Setup (Alternative to Maven)**

#### **Gradle Installation**
```bash
# Windows - Using Chocolatey
choco install gradle

# macOS - Using Homebrew
brew install gradle

# Linux - Using SDKMAN (recommended)
sdk install gradle 8.4

# Verify installation
gradle -version
```

#### **Gradle Configuration**

**Global Properties (`~/.gradle/gradle.properties`):**
```properties
# JVM settings
org.gradle.jvmargs=-Xmx2048m -XX:MaxMetaspaceSize=512m

# Parallel builds
org.gradle.parallel=true

# Build cache
org.gradle.caching=true

# Configure only on demand
org.gradle.configureondemand=true

# Gradle daemon
org.gradle.daemon=true
```

#### **Essential Gradle Commands**
```bash
# Project tasks
./gradlew clean              # Clean build directory
./gradlew build              # Full build
./gradlew test               # Run tests
./gradlew bootRun            # Run Spring Boot application
./gradlew bootJar            # Create executable JAR

# Dependency management
./gradlew dependencies       # Show dependencies
./gradlew dependencyInsight --dependency spring-boot-starter-web

# Build analysis
./gradlew build --scan       # Generate build scan
./gradlew tasks              # List available tasks
./gradlew projects          # List sub-projects
```

#### **Sample Gradle Build File (`build.gradle`)**
```gradle
plugins {
    id 'org.springframework.boot' version '3.2.0'
    id 'io.spring.dependency-management' version '1.1.4'
    id 'java'
}

group = 'com.example'
version = '0.0.1-SNAPSHOT'

java {
    sourceCompatibility = '17'
}

configurations {
    compileOnly {
        extendsFrom annotationProcessor
    }
}

repositories {
    mavenCentral()
}

dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-web'
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
    implementation 'org.springframework.boot:spring-boot-starter-security'
    implementation 'org.springframework.boot:spring-boot-starter-actuator'
    
    compileOnly 'org.projectlombok:lombok'
    developmentOnly 'org.springframework.boot:spring-boot-devtools'
    
    runtimeOnly 'com.h2database:h2'
    
    annotationProcessor 'org.projectlombok:lombok'
    
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
    testImplementation 'org.springframework.security:spring-security-test'
}

tasks.named('test') {
    useJUnitPlatform()
}

// Custom tasks
task printVersion {
    doLast {
        println "Project version: $version"
    }
}
```

---

## 🔄 Git Version Control Setup

### **Git Installation and Configuration**

#### **Git Installation**
```bash
# Windows - Using Chocolatey
choco install git

# macOS - Using Homebrew
brew install git

# Linux - Package manager
sudo apt install git-all  # Ubuntu/Debian
sudo yum install git-all  # CentOS/RHEL

# Verify installation
git --version
```

#### **Initial Git Configuration**
```bash
# Global configuration
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"

# Editor configuration
git config --global core.editor "code --wait"  # VS Code
# or
git config --global core.editor "idea --wait"  # IntelliJ IDEA

# Line ending configuration
git config --global core.autocrlf true   # Windows
git config --global core.autocrlf input  # macOS/Linux

# Default branch name
git config --global init.defaultBranch main

# Credential helper
git config --global credential.helper cache  # Linux
git config --global credential.helper store  # Persistent storage

# Show current configuration
git config --list
```

### **Git Workflow for Spring Boot Projects**

#### **Repository Initialization**
```bash
# Initialize new repository
git init
git add .
git commit -m "Initial commit: Spring Boot project setup"

# Connect to remote repository
git remote add origin https://github.com/username/repository.git
git branch -M main
git push -u origin main
```

#### **Branching Strategy**
```bash
# Feature branch workflow
git checkout -b feature/user-authentication
# Develop feature
git add .
git commit -m "feat: implement user authentication with JWT"
git push origin feature/user-authentication
# Create pull request
# Merge to main

# Gitflow workflow
git flow init
git flow feature start user-authentication
# Develop feature
git flow feature finish user-authentication

# Release workflow
git flow release start 1.0.0
# Final preparations
git flow release finish 1.0.0
```

#### **Essential Git Commands for Spring Boot**
```bash
# Daily workflow
git status                   # Check working directory status
git add .                    # Stage all changes
git add -p                   # Stage changes interactively
git commit -m "message"      # Commit with message
git push                     # Push to remote
git pull                     # Pull from remote

# Branch management
git branch                   # List branches
git checkout -b feature-name # Create and switch to branch
git merge feature-name       # Merge branch
git branch -d feature-name   # Delete branch

# History and inspection
git log --oneline           # Compact log
git log --graph --oneline   # Visual branch history
git diff                    # Show unstaged changes
git diff --staged           # Show staged changes
git show HEAD               # Show last commit

# Undo operations
git reset HEAD~1            # Undo last commit (keep changes)
git reset --hard HEAD~1     # Undo last commit (discard changes)
git revert HEAD            # Create new commit that undoes changes
git checkout -- filename   # Discard changes to file
```

### **Git Best Practices for Spring Boot**

#### **.gitignore Configuration**
```gitignore
# Compiled class files
*.class

# Log files
*.log

# BlueJ files
*.ctxt

# Mobile Tools for Java (J2ME)
.mtj.tmp/

# Package Files
*.jar
*.war
*.nar
*.ear
*.zip
*.tar.gz
*.rar

# Virtual machine crash logs
hs_err_pid*

# Maven
target/
!.mvn/wrapper/maven-wrapper.jar
!**/src/main/**/target/
!**/src/test/**/target/

# Gradle
.gradle/
build/
!gradle/wrapper/gradle-wrapper.jar
!**/src/main/**/build/
!**/src/test/**/build/

# IntelliJ IDEA
.idea/
*.iws
*.iml
*.ipr
out/
!**/src/main/**/out/
!**/src/test/**/out/

# Eclipse
.apt_generated
.classpath
.factorypath
.project
.settings
.springBeans
.sts4-cache
bin/
!**/src/main/**/bin/
!**/src/test/**/bin/

# VS Code
.vscode/

# OS generated files
.DS_Store
.DS_Store?
._*
.Spotlight-V100
.Trashes
ehthumbs.db
Thumbs.db

# Spring Boot
!**/src/main/**
!**/src/test/**

# Application specific
application-local.properties
application-dev.properties
application-secrets.yml
*.env
```

#### **Commit Message Conventions**
```bash
# Conventional Commits format
<type>[optional scope]: <description>

[optional body]

[optional footer(s)]

# Types:
feat:     # New feature
fix:      # Bug fix
docs:     # Documentation only changes
style:    # Code style changes (formatting, etc.)
refactor: # Code refactoring
test:     # Adding missing tests
chore:    # Maintenance tasks

# Examples:
feat(auth): implement JWT authentication
fix(db): resolve connection timeout issue
docs(api): update REST endpoint documentation
refactor(service): extract common validation logic
test(controller): add integration tests for user endpoints
chore(deps): update Spring Boot to 3.2.0
```

---

## 🐳 Docker Setup

### **Docker Installation**

#### **Docker Desktop Installation**
```bash
# Windows
# Download from https://desktop.docker.com/win/main/amd64/Docker%20Desktop%20Installer.exe
# Run installer and restart

# macOS
brew install --cask docker
# or download from https://desktop.docker.com/mac/main/amd64/Docker.dmg

# Linux (Ubuntu)
sudo apt update
sudo apt install apt-transport-https ca-certificates curl software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu focal stable"
sudo apt update
sudo apt install docker-ce

# Start Docker service
sudo systemctl start docker
sudo systemctl enable docker

# Add user to docker group
sudo usermod -aG docker ${USER}

# Verify installation
docker --version
docker run hello-world
```

### **Docker for Spring Boot Development**

#### **Dockerfile for Spring Boot**
```dockerfile
# Multi-stage build for optimal image size
FROM openjdk:17-jdk-slim as builder

# Set working directory
WORKDIR /app

# Copy Maven wrapper and pom.xml
COPY .mvn/ .mvn
COPY mvnw pom.xml ./

# Download dependencies (cached layer)
RUN ./mvnw dependency:go-offline

# Copy source code
COPY src ./src

# Build application
RUN ./mvnw clean package -DskipTests

# Production stage
FROM openjdk:17-jre-slim

# Create app user
RUN addgroup --system spring && adduser --system spring --ingroup spring

# Set working directory
WORKDIR /app

# Copy JAR from builder stage
COPY --from=builder /app/target/*.jar app.jar

# Change ownership
RUN chown spring:spring app.jar

# Switch to app user
USER spring:spring

# Expose port
EXPOSE 8080

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=60s --retries=3 \
  CMD curl -f http://localhost:8080/actuator/health || exit 1

# Run application
ENTRYPOINT ["java", "-jar", "app.jar"]
```

#### **Docker Compose for Development Environment**
```yaml
# docker-compose.yml
version: '3.8'

services:
  # Spring Boot application
  app:
    build: .
    ports:
      - "8080:8080"
    environment:
      - SPRING_PROFILES_ACTIVE=docker
      - SPRING_DATASOURCE_URL=jdbc:postgresql://db:5432/springboot_db
      - SPRING_DATASOURCE_USERNAME=springboot
      - SPRING_DATASOURCE_PASSWORD=password
    depends_on:
      db:
        condition: service_healthy
    volumes:
      - ./logs:/app/logs
    networks:
      - spring-network

  # PostgreSQL database
  db:
    image: postgres:15-alpine
    restart: always
    environment:
      POSTGRES_DB: springboot_db
      POSTGRES_USER: springboot
      POSTGRES_PASSWORD: password
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./sql/init.sql:/docker-entrypoint-initdb.d/init.sql
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U springboot"]
      interval: 10s
      timeout: 5s
      retries: 5
    networks:
      - spring-network

  # Redis for caching
  redis:
    image: redis:7-alpine
    restart: always
    ports:
      - "6379:6379"
    command: redis-server --appendonly yes
    volumes:
      - redis_data:/data
    networks:
      - spring-network

  # Adminer for database management
  adminer:
    image: adminer:latest
    restart: always
    ports:
      - "8081:8080"
    depends_on:
      - db
    networks:
      - spring-network

volumes:
  postgres_data:
  redis_data:

networks:
  spring-network:
    driver: bridge
```

#### **Essential Docker Commands for Spring Boot**
```bash
# Build image
docker build -t my-spring-app .

# Run container
docker run -p 8080:8080 my-spring-app

# Run with environment variables
docker run -p 8080:8080 -e SPRING_PROFILES_ACTIVE=prod my-spring-app

# Run in background
docker run -d -p 8080:8080 --name spring-app my-spring-app

# View logs
docker logs spring-app
docker logs -f spring-app  # Follow logs

# Execute commands in container
docker exec -it spring-app bash

# Docker Compose commands
docker-compose up                    # Start all services
docker-compose up -d                 # Start in background
docker-compose down                  # Stop and remove containers
docker-compose logs app              # View app logs
docker-compose exec app bash        # Execute command in app container

# Clean up
docker container prune               # Remove stopped containers
docker image prune                   # Remove unused images
docker volume prune                  # Remove unused volumes
docker system prune                  # Remove all unused resources
```

---

## ⚙️ Development Tools and Productivity

### **API Testing Tools**

#### **Postman Setup**
```bash
# Install Postman
# Windows/macOS: Download from https://www.postman.com/downloads/
# Linux: Snap package
sudo snap install postman

# Alternative: Insomnia
sudo snap install insomnia
```

#### **HTTP Client Files in IntelliJ**
```http
### Test user registration
POST http://localhost:8080/api/users/register
Content-Type: application/json

{
  "username": "john_doe",
  "email": "john@example.com",
  "password": "securePassword123"
}

### Test user login
POST http://localhost:8080/api/auth/login
Content-Type: application/json

{
  "username": "john_doe",
  "password": "securePassword123"
}

> {%
client.test("Request executed successfully", function() {
    client.assert(response.status === 200, "Response status is not 200");
});
client.global.set("auth_token", response.body.token);
%}

### Test authenticated endpoint
GET http://localhost:8080/api/users/profile
Authorization: Bearer {{auth_token}}

### Test health endpoint
GET http://localhost:8080/actuator/health
```

#### **curl Commands for API Testing**
```bash
# Test REST endpoints
curl -X GET http://localhost:8080/api/users
curl -X POST http://localhost:8080/api/users \
  -H "Content-Type: application/json" \
  -d '{"name":"John Doe","email":"john@example.com"}'

# Test with authentication
curl -X GET http://localhost:8080/api/protected \
  -H "Authorization: Bearer YOUR_JWT_TOKEN"

# Test file upload
curl -X POST http://localhost:8080/api/upload \
  -F "file=@document.pdf" \
  -F "description=Important document"

# Test actuator endpoints
curl http://localhost:8080/actuator/health
curl http://localhost:8080/actuator/metrics
curl http://localhost:8080/actuator/info
```

### **Database Tools**

#### **Database GUI Clients**
```bash
# DBeaver (Free, Universal)
# Download from https://dbeaver.io/download/

# IntelliJ IDEA Database Tools (Built-in)
# Already included with IntelliJ IDEA

# pgAdmin (PostgreSQL)
# Download from https://www.pgadmin.org/download/

# MySQL Workbench (MySQL)
# Download from https://www.mysql.com/products/workbench/
```

#### **Database Connection Examples**
```yaml
# application-dev.yml
spring:
  datasource:
    url: jdbc:h2:mem:testdb
    driver-class-name: org.h2.Driver
    username: sa
    password: password
  
  h2:
    console:
      enabled: true
      path: /h2-console

---
# application-prod.yml
spring:
  datasource:
    url: jdbc:postgresql://localhost:5432/springboot_db
    driver-class-name: org.postgresql.Driver
    username: ${DB_USERNAME:springboot}
    password: ${DB_PASSWORD:password}
  
  jpa:
    hibernate:
      ddl-auto: validate
    show-sql: false
```

### **Monitoring and Profiling Tools**

#### **Application Monitoring**
```bash
# VisualVM (Free JVM profiler)
# Download from https://visualvm.github.io/

# JProfiler (Commercial)
# Download from https://www.ej-technologies.com/products/jprofiler/overview.html

# YourKit (Commercial)
# Download from https://www.yourkit.com/java/profiler/

# Built-in tools
jstack <pid>           # Thread dump
jmap -histo <pid>      # Heap histogram
jstat -gc <pid>        # Garbage collection stats
```

#### **JVM Monitoring Configuration**
```bash
# JVM parameters for monitoring
-Dcom.sun.management.jmxremote
-Dcom.sun.management.jmxremote.port=9999
-Dcom.sun.management.jmxremote.authenticate=false
-Dcom.sun.management.jmxremote.ssl=false

# Flight Recorder
-XX:+FlightRecorder
-XX:StartFlightRecording=duration=60s,filename=recording.jfr

# GC logging
-Xlog:gc*:gc.log:time,tags
```

---

## 🚀 Development Workflow Setup

### **Project Template Creation**

#### **Spring Boot Project Template Structure**
```
my-spring-boot-template/
├── .gitignore
├── .editorconfig
├── README.md
├── docker-compose.yml
├── Dockerfile
├── pom.xml
├── src/
│   ├── main/
│   │   ├── java/
│   │   │   └── com/example/template/
│   │   │       ├── TemplateApplication.java
│   │   │       ├── config/
│   │   │       │   ├── ApplicationConfig.java
│   │   │       │   ├── SecurityConfig.java
│   │   │       │   └── DatabaseConfig.java
│   │   │       ├── controller/
│   │   │       │   └── HealthController.java
│   │   │       ├── service/
│   │   │       ├── repository/
│   │   │       ├── domain/
│   │   │       ├── dto/
│   │   │       ├── exception/
│   │   │       │   ├── GlobalExceptionHandler.java
│   │   │       │   └── BusinessException.java
│   │   │       └── util/
│   │   └── resources/
│   │       ├── application.yml
│   │       ├── application-dev.yml
│   │       ├── application-prod.yml
│   │       ├── logback-spring.xml
│   │       ├── static/
│   │       └── templates/
│   └── test/
│       ├── java/
│       │   └── com/example/template/
│       │       ├── TemplateApplicationTests.java
│       │       ├── controller/
│       │       ├── service/
│       │       └── repository/
│       └── resources/
│           └── application-test.yml
├── docs/
│   ├── api/
│   ├── deployment/
│   └── development.md
└── scripts/
    ├── build.sh
    ├── deploy.sh
    └── local-setup.sh
```

#### **EditorConfig Setup**
```ini
# .editorconfig
root = true

[*]
charset = utf-8
end_of_line = lf
indent_style = space
indent_size = 4
insert_final_newline = true
trim_trailing_whitespace = true

[*.yml]
indent_size = 2

[*.xml]
indent_size = 2

[*.json]
indent_size = 2

[*.md]
trim_trailing_whitespace = false
```

### **Automated Setup Scripts**

#### **Development Setup Script (setup.sh)**
```bash
#!/bin/bash

echo "🚀 Setting up Spring Boot development environment..."

# Check prerequisites
check_command() {
    if ! command -v $1 &> /dev/null; then
        echo "❌ $1 is not installed. Please install it first."
        exit 1
    else
        echo "✅ $1 is available"
    fi
}

echo "Checking prerequisites..."
check_command java
check_command mvn
check_command git
check_command docker

# Create project directory
PROJECT_NAME=${1:-my-spring-boot-app}
echo "Creating project: $PROJECT_NAME"
mkdir -p $PROJECT_NAME
cd $PROJECT_NAME

# Initialize Git repository
git init
git config user.name "Developer"
git config user.email "developer@example.com"

# Download project template
curl -s https://start.spring.io/starter.zip \
  -d dependencies=web,data-jpa,security,actuator \
  -d type=maven-project \
  -d language=java \
  -d bootVersion=3.2.0 \
  -d baseDir=$PROJECT_NAME \
  -d groupId=com.example \
  -d artifactId=$PROJECT_NAME \
  -d name=$PROJECT_NAME \
  -d description="Spring Boot Application" \
  -d packageName=com.example.${PROJECT_NAME//-/} \
  -d packaging=jar \
  -d javaVersion=17 \
  -o project.zip

unzip project.zip
rm project.zip

# Setup Docker environment
cat > docker-compose.yml << EOF
version: '3.8'
services:
  db:
    image: postgres:15-alpine
    environment:
      POSTGRES_DB: ${PROJECT_NAME}_db
      POSTGRES_USER: developer
      POSTGRES_PASSWORD: password
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
EOF

# Initial commit
git add .
git commit -m "Initial project setup"

echo "✅ Project $PROJECT_NAME created successfully!"
echo "📁 cd $PROJECT_NAME"
echo "🏃 mvn spring-boot:run"
```

#### **Local Development Script (dev.sh)**
```bash
#!/bin/bash

# Local development helper script

case "$1" in
    start)
        echo "🚀 Starting development environment..."
        docker-compose up -d
        echo "⏳ Waiting for database..."
        sleep 10
        mvn spring-boot:run -Dspring-boot.run.profiles=dev
        ;;
    stop)
        echo "🛑 Stopping development environment..."
        docker-compose down
        ;;
    test)
        echo "🧪 Running tests..."
        mvn clean test
        ;;
    build)
        echo "🔨 Building application..."
        mvn clean package
        ;;
    clean)
        echo "🧹 Cleaning up..."
        mvn clean
        docker-compose down -v
        docker system prune -f
        ;;
    logs)
        echo "📋 Showing application logs..."
        docker-compose logs -f
        ;;
    *)
        echo "Usage: $0 {start|stop|test|build|clean|logs}"
        exit 1
        ;;
esac
```

---

## 📋 Environment Verification Checklist

### **Development Environment Checklist**

```bash
# ✅ Java Development Kit
[ ] Java 17+ installed and configured
[ ] JAVA_HOME environment variable set
[ ] Multiple JDK versions managed (SDKMAN/jenv)

# ✅ Build Tools
[ ] Maven 3.8+ installed and configured
[ ] Local repository configured (~/.m2/settings.xml)
[ ] Gradle 8+ installed (optional)

# ✅ IDE Setup
[ ] IntelliJ IDEA installed with Spring plugins
[ ] Code style and live templates configured
[ ] Project import working correctly

# ✅ Version Control
[ ] Git installed and configured
[ ] SSH keys generated and added to GitHub/GitLab
[ ] .gitignore properly configured

# ✅ Containerization
[ ] Docker Desktop installed and running
[ ] Docker Compose available
[ ] Can build and run Spring Boot containers

# ✅ Database Tools
[ ] Database client installed (DBeaver/IntelliJ)
[ ] Connection to local databases working
[ ] Docker database containers working

# ✅ API Testing
[ ] Postman or similar tool installed
[ ] HTTP client files working in IDE
[ ] curl commands working

# ✅ Monitoring
[ ] Can access Spring Boot Actuator endpoints
[ ] JVM monitoring tools available
[ ] Log files accessible and readable
```

### **Quick Verification Commands**
```bash
# Java environment
java -version
javac -version
echo $JAVA_HOME

# Build tools
mvn -version
gradle -version  # if using Gradle

# Git configuration
git --version
git config --list

# Docker
docker --version
docker-compose --version
docker run hello-world

# Spring Boot quick test
spring init --dependencies=web test-app
cd test-app
./mvnw spring-boot:run
curl http://localhost:8080/actuator/health
```

---

## 🔧 Troubleshooting Common Issues

### **Java Issues**
```bash
# JAVA_HOME not set
export JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64  # Linux
export JAVA_HOME=/Library/Java/JavaVirtualMachines/openjdk-17.jdk/Contents/Home  # macOS

# Multiple Java versions conflict
sudo update-alternatives --config java  # Linux
brew unlink java && brew link openjdk@17  # macOS

# Classpath issues
mvn clean compile  # Rebuild project
./gradlew clean build  # Gradle rebuild
```

### **Maven Issues**
```bash
# Dependency resolution problems
mvn dependency:purge-local-repository
mvn clean install -U  # Update snapshots

# Build failures
mvn clean compile -X  # Debug mode
mvn clean install -DskipTests  # Skip tests temporarily

# Plugin version conflicts
mvn versions:display-plugin-updates
mvn versions:display-dependency-updates
```

### **Docker Issues**
```bash
# Docker daemon not running
systemctl start docker  # Linux
# Start Docker Desktop manually on Windows/macOS

# Permission denied
sudo usermod -aG docker $USER  # Add user to docker group
newgrp docker  # Refresh group membership

# Container port conflicts
docker ps  # Check running containers
docker kill $(docker ps -q)  # Stop all containers

# Image build failures
docker build --no-cache -t my-app .  # Disable cache
docker system prune -f  # Clean up
```

### **IntelliJ IDEA Issues**
```bash
# Project import problems
File → Invalidate Caches and Restart
Delete .idea folder and reimport project

# Maven/Gradle sync issues
View → Tool Windows → Maven → Refresh
View → Tool Windows → Gradle → Refresh

# Plugin conflicts
File → Settings → Plugins → Disable conflicting plugins
Restart IDE
```

---

## 📚 Summary and Next Steps

### **Environment Setup Summary**

You now have a complete Spring Boot development environment with:
- ✅ **Java 17+** with proper JDK management
- ✅ **IntelliJ IDEA** with Spring Boot plugins
- ✅ **Maven/Gradle** for build automation
- ✅ **Git** for version control
- ✅ **Docker** for containerization
- ✅ **Database tools** for data management
- ✅ **API testing tools** for endpoint verification
- ✅ **Monitoring tools** for application observability

### **What's Next?**

#### **Chapter 3: Spring Boot Fundamentals**
- Deep dive into auto-configuration mechanisms
- Understanding starter dependencies and their purposes
- Configuration properties and profiles management
- Application events and lifecycle hooks

#### **Preparation Tasks**
1. Create your first Spring Boot project using Spring Initializr
2. Import the project into IntelliJ IDEA
3. Run the application and verify actuator endpoints
4. Set up a simple Docker container for the application
5. Create a Git repository and make your first commit

### **Practice Exercises**

#### **Exercise 1: Complete Environment Setup**
1. Install all required tools following this guide
2. Verify each installation with the provided commands
3. Complete the environment verification checklist

#### **Exercise 2: Project Creation**
1. Use Spring Initializr to create a new project
2. Import into IntelliJ IDEA
3. Add a simple REST endpoint
4. Run and test the application

#### **Exercise 3: Containerization**
1. Create a Dockerfile for your Spring Boot app
2. Build and run the Docker container
3. Set up docker-compose with a database
4. Verify the application works in containers

#### **Exercise 4: Version Control**
1. Initialize a Git repository
2. Configure .gitignore properly
3. Make meaningful commits with conventional messages
4. Push to a remote repository (GitHub/GitLab)

---

**🎯 Your development environment is ready! Let's dive into Spring Boot fundamentals!**

**[Next Chapter: Spring Boot Fundamentals →](./03-spring-boot-fundamentals.md)**

---

*Duration: ~2 hours | Difficulty: Beginner | Prerequisites: Basic command line knowledge*
