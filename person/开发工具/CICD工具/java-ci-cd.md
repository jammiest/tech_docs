# Java 项目 CI/CD

## 1. 概述

Java 项目的 CI/CD 流程需要处理复杂的依赖管理、多模块构建和 JVM 特性。本指南提供完整的 Java 项目 CI/CD 实施方案，涵盖 Maven/Gradle 构建、测试、代码质量检查、容器化和部署全流程。

## 2. 环境配置

### 2.1 基础环境设置
```bash
#!/bin/bash
# setup-java-environment.sh

# 安装 JDK
sudo apt update
sudo apt install -y openjdk-17-jdk openjdk-17-jre

# 配置环境变量
echo 'export JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64' >> ~/.bashrc
echo 'export PATH=$JAVA_HOME/bin:$PATH' >> ~/.bashrc
source ~/.bashrc

# 安装构建工具
wget https://archive.apache.org/dist/maven/maven-3/3.9.4/binaries/apache-maven-3.9.4-bin.tar.gz
tar -xzf apache-maven-3.9.4-bin.tar.gz
sudo mv apache-maven-3.9.4 /opt/maven
echo 'export PATH=/opt/maven/bin:$PATH' >> ~/.bashrc

# 安装常用工具
sudo apt install -y git curl unzip

# 验证安装
java -version
mvn -version
```

### 2.2 Docker 化 Java 环境
```dockerfile
# Dockerfile
# 构建阶段
FROM maven:3.9.4-eclipse-temurin-17 AS builder

WORKDIR /app

# 复制依赖文件
COPY pom.xml .
RUN mvn dependency:go-offline -B

# 复制源代码
COPY src ./src

# 构建应用
RUN mvn package -DskipTests -Dmaven.test.skip=true

# 最终阶段
FROM eclipse-temurin:17-jre

WORKDIR /app

# 从构建阶段复制JAR文件
COPY --from=builder /app/target/*.jar /app/app.jar

# 创建非root用户
RUN groupadd -r javaapp && useradd -r -g javaapp javaapp
USER javaapp

# 暴露端口
EXPOSE 8080

# 启动命令
ENTRYPOINT ["java", "-jar", "/app/app.jar"]
```

## 3. CI/CD 流水线设计

### 3.1 GitHub Actions 配置
```yaml
# .github/workflows/java-ci-cd.yml
name: Java CI/CD Pipeline

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

env:
  JAVA_VERSION: '17'
  MAVEN_VERSION: '3.9.4'
  DOCKER_IMAGE: 'ghcr.io/${{ github.repository_owner }}/${{ github.event.repository.name }}'

jobs:
  code-quality:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Java
        uses: actions/setup-java@v3
        with:
          java-version: ${{ env.JAVA_VERSION }}
          distribution: 'temurin'
          cache: 'maven'
      
      - name: Run Checkstyle
        run: mvn checkstyle:check
      
      - name: Run PMD
        run: mvn pmd:check
      
      - name: Run SpotBugs
        run: mvn spotbugs:check

  test:
    runs-on: ubuntu-latest
    needs: code-quality
    services:
      postgres:
        image: postgres:15
        env:
          POSTGRES_PASSWORD: postgres
          POSTGRES_DB: testdb
        ports:
          - 5432:5432
        options: --health-cmd="pg_isready" --health-interval=10s --health-timeout=5s --health-retries=3
      
      redis:
        image: redis:7
        ports:
          - 6379:6379
        options: --health-cmd="redis-cli ping" --health-interval=10s --health-timeout=5s --health-retries=3
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Java
        uses: actions/setup-java@v3
        with:
          java-version: ${{ env.JAVA_VERSION }}
          distribution: 'temurin'
          cache: 'maven'
      
      - name: Run tests
        run: mvn test -Dspring.profiles.active=test
      
      - name: Upload test results
        uses: actions/upload-artifact@v3
        with:
          name: test-results
          path: target/surefire-reports/

  build:
    runs-on: ubuntu-latest
    needs: test
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Java
        uses: actions/setup-java@v3
        with:
          java-version: ${{ env.JAVA_VERSION }}
          distribution: 'temurin'
          cache: 'maven'
      
      - name: Build package
        run: mvn package -DskipTests -Dmaven.test.skip=true
      
      - name: Upload artifact
        uses: actions/upload-artifact@v3
        with:
          name: java-package
          path: target/*.jar

  docker:
    runs-on: ubuntu-latest
    needs: build
    steps:
      - uses: actions/checkout@v4
      
      - name: Download artifact
        uses: actions/download-artifact@v3
        with:
          name: java-package
          path: target
      
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v2
      
      - name: Login to GitHub Container Registry
        uses: docker/login-action@v2
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
      
      - name: Build and push
        uses: docker/build-push-action@v4
        with:
          context: .
          push: ${{ github.ref == 'refs/heads/main' }}
          tags: ${{ env.DOCKER_IMAGE }}:${{ github.sha }}
          labels: |
            org.opencontainers.image.source=${{ github.repository }}
            org.opencontainers.image.revision=${{ github.sha }}

  deploy:
    runs-on: ubuntu-latest
    needs: docker
    if: github.ref == 'refs/heads/main'
    environment: production
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Install kubectl
        uses: azure/setup-kubectl@v3
        with:
          version: 'latest'
      
      - name: Configure Kubernetes
        run: |
          mkdir -p ~/.kube
          echo "${{ secrets.KUBE_CONFIG }}" > ~/.kube/config
          kubectl config use-context ${{ secrets.KUBE_CONTEXT }}
      
      - name: Deploy to Kubernetes
        run: |
          kubectl set image deployment/myapp myapp=${{ env.DOCKER_IMAGE }}:${{ github.sha }} -n production
          kubectl rollout status deployment/myapp -n production
```

### 3.2 GitLab CI 配置
```yaml
# .gitlab-ci.yml
image: maven:3.9.4-eclipse-temurin-17

stages:
  - code_quality
  - test
  - build
  - deploy

variables:
  MAVEN_OPTS: "-Dmaven.repo.local=.m2/repository"
  MAVEN_CLI_OPTS: "--batch-mode --errors --fail-fast --show-version"

before_script:
  - java -version
  - mvn -version

code_quality:
  stage: code_quality
  script:
    - mvn $MAVEN_CLI_OPTS checkstyle:check
    - mvn $MAVEN_CLI_OPTS pmd:check
    - mvn $MAVEN_CLI_OPTS spotbugs:check

test:
  stage: test
  services:
    - postgres:15
    - redis:7
  variables:
    POSTGRES_PASSWORD: "postgres"
    POSTGRES_DB: "testdb"
    REDIS_PASSWORD: "redis"
    SPRING_PROFILES_ACTIVE: "test"
  script:
    - mvn $MAVEN_CLI_OPTS test
  artifacts:
    paths:
      - target/surefire-reports/
    expire_in: 1 week

build:
  stage: build
  script:
    - mvn $MAVEN_CLI_OPTS package -DskipTests
  artifacts:
    paths:
      - target/*.jar
    expire_in: 1 week

docker-build:
  stage: build
  image: docker:20.10
  services:
    - docker:20.10-dind
  variables:
    DOCKER_BUILDKIT: 1
  script:
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
    - docker build -t $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA .
    - docker push $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
  only:
    - main

deploy-production:
  stage: deploy
  image: bitnami/kubectl:latest
  environment: production
  only:
    - main
  script:
    - kubectl config use-context $KUBE_CONTEXT
    - kubectl set image deployment/myapp myapp=$CI_REGISTRY_IMAGE:$CI_COMMIT_SHA -n production
    - kubectl rollout status deployment/myapp -n production
```

## 4. 代码质量工具

### 4.1 Maven 插件配置
```xml
<!-- pom.xml 代码质量配置 -->
<build>
    <plugins>
        <!-- Checkstyle 配置 -->
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-checkstyle-plugin</artifactId>
            <version>3.3.0</version>
            <configuration>
                <configLocation>google_checks.xml</configLocation>
                <includeTestSourceDirectory>true</includeTestSourceDirectory>
            </configuration>
            <executions>
                <execution>
                    <phase>verify</phase>
                    <goals>
                        <goal>check</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>

        <!-- PMD 配置 -->
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-pmd-plugin</artifactId>
            <version>3.21.0</version>
            <configuration>
                <rulesets>
                    <ruleset>category/java/errorprone.xml</ruleset>
                    <ruleset>category/java/bestpractices.xml</ruleset>
                </rulesets>
            </configuration>
            <executions>
                <execution>
                    <phase>verify</phase>
                    <goals>
                        <goal>check</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>

        <!-- SpotBugs 配置 -->
        <plugin>
            <groupId>com.github.spotbugs</groupId>
            <artifactId>spotbugs-maven-plugin</artifactId>
            <version>4.7.3.6</version>
            <configuration>
                <effort>Max</effort>
                <threshold>Low</threshold>
            </configuration>
            <executions>
                <execution>
                    <phase>verify</phase>
                    <goals>
                        <goal>check</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>

        <!-- JaCoCo 代码覆盖率 -->
        <plugin>
            <groupId>org.jacoco</groupId>
            <artifactId>jacoco-maven-plugin</artifactId>
            <version>0.8.10</version>
            <executions>
                <execution>
                    <goals>
                        <goal>prepare-agent</goal>
                    </goals>
                </execution>
                <execution>
                    <id>report</id>
                    <phase>test</phase>
                    <goals>
                        <goal>report</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

### 4.2 代码格式化配置
```xml
<!-- checkstyle.xml -->
<?xml version="1.0"?>
<!DOCTYPE module PUBLIC
        "-//Checkstyle//DTD Checkstyle Configuration 1.3//EN"
        "https://checkstyle.org/dtds/configuration_1_3.dtd">
<module name="Checker">
    <module name="TreeWalker">
        <!-- 导入检查 -->
        <module name="AvoidStarImport"/>
        <module name="RedundantImport"/>
        <module name="UnusedImports"/>

        <!-- 代码风格 -->
        <module name="LeftCurly"/>
        <module name="RightCurly"/>
        <module name="NeedBraces"/>
        <module name="EmptyStatement"/>
        <module name="EqualsHashCode"/>

        <!-- 命名约定 -->
        <module name="ConstantName"/>
        <module name="LocalFinalVariableName"/>
        <module name="LocalVariableName"/>
        <module name="MemberName"/>
        <module name="MethodName"/>
        <module name="PackageName"/>
        <module name="ParameterName"/>
        <module name="StaticVariableName"/>
        <module name="TypeName"/>

        <!-- 其他检查 -->
        <module name="ArrayTypeStyle"/>
        <module name="ModifierOrder"/>
        <module name="VisibilityModifier"/>
    </module>

    <!-- 文件长度检查 -->
    <module name="FileLength">
        <property name="max" value="2000"/>
    </module>

    <!-- 行长度检查 -->
    <module name="LineLength">
        <property name="max" value="120"/>
    </module>
</module>
```

## 5. 测试策略

### 5.1 单元测试最佳实践
```java
// UserServiceTest.java
@ExtendWith(MockitoExtension.class)
class UserServiceTest {

    @Mock
    private UserRepository userRepository;

    @InjectMocks
    private UserService userService;

    @Test
    void getUserById_WhenUserExists_ReturnsUser() {
        // Arrange
        Long userId = 1L;
        User expectedUser = new User(userId, "john@example.com", "John Doe");
        when(userRepository.findById(userId)).thenReturn(Optional.of(expectedUser));

        // Act
        User result = userService.getUserById(userId);

        // Assert
        assertNotNull(result);
        assertEquals(expectedUser, result);
        verify(userRepository).findById(userId);
    }

    @Test
    void getUserById_WhenUserNotExists_ThrowsException() {
        // Arrange
        Long userId = 999L;
        when(userRepository.findById(userId)).thenReturn(Optional.empty());

        // Act & Assert
        assertThrows(UserNotFoundException.class, () -> userService.getUserById(userId));
        verify(userRepository).findById(userId);
    }

    @Test
    void createUser_WithValidData_ReturnsCreatedUser() {
        // Arrange
        CreateUserRequest request = new CreateUserRequest("john@example.com", "John Doe");
        User savedUser = new User(1L, "john@example.com", "John Doe");
        when(userRepository.save(any(User.class))).thenReturn(savedUser);

        // Act
        User result = userService.createUser(request);

        // Assert
        assertNotNull(result);
        assertEquals("john@example.com", result.getEmail());
        verify(userRepository).save(any(User.class));
    }

    @ParameterizedTest
    @ValueSource(strings = {"", "invalid-email", "no@domain"})
    void createUser_WithInvalidEmail_ThrowsException(String invalidEmail) {
        // Arrange
        CreateUserRequest request = new CreateUserRequest(invalidEmail, "John Doe");

        // Act & Assert
        assertThrows(InvalidEmailException.class, () -> userService.createUser(request));
        verify(userRepository, never()).save(any(User.class));
    }
}
```

### 5.2 集成测试配置
```java
// UserIntegrationTest.java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@Testcontainers
@ActiveProfiles("test")
class UserIntegrationTest {

    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:15")
            .withDatabaseName("testdb")
            .withUsername("test")
            .withPassword("test");

    @Container
    static RedisContainer redis = new RedisContainer("redis:7")
            .withExposedPorts(6379);

    @DynamicPropertySource
    static void configureProperties(DynamicPropertyRegistry registry) {
        registry.add("spring.datasource.url", postgres::getJdbcUrl);
        registry.add("spring.datasource.username", postgres::getUsername);
        registry.add("spring.datasource.password", postgres::getPassword);
        registry.add("spring.redis.host", redis::getHost);
        registry.add("spring.redis.port", redis::getFirstMappedPort);
    }

    @Autowired
    private TestRestTemplate restTemplate;

    @Autowired
    private UserRepository userRepository;

    @BeforeEach
    void setUp() {
        userRepository.deleteAll();
    }

    @Test
    void createUser_ThroughApi_ReturnsCreatedUser() {
        // Arrange
        CreateUserRequest request = new CreateUserRequest("test@example.com", "Test User");

        // Act
        ResponseEntity<UserResponse> response = restTemplate.postForEntity(
                "/api/users", request, UserResponse.class);

        // Assert
        assertEquals(HttpStatus.CREATED, response.getStatusCode());
        assertNotNull(response.getBody());
        assertEquals("test@example.com", response.getBody().email());

        // Verify database
        List<User> users = userRepository.findAll();
        assertEquals(1, users.size());
        assertEquals("test@example.com", users.get(0).getEmail());
    }
}
```

## 6. 部署策略

### 6.1 Kubernetes 部署配置
```yaml
# k8s/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: java-app
  namespace: production
  labels:
    app: java-app
    environment: production
spec:
  replicas: 3
  selector:
    matchLabels:
      app: java-app
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  template:
    metadata:
      labels:
        app: java-app
        environment: production
      annotations:
        prometheus.io/scrape: "true"
        prometheus.io/port: "8080"
    spec:
      containers:
      - name: java-app
        image: ghcr.io/myorg/java-app:latest
        imagePullPolicy: Always
        ports:
        - containerPort: 8080
        livenessProbe:
          httpGet:
            path: /actuator/health/liveness
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
          timeoutSeconds: 5
        readinessProbe:
          httpGet:
            path: /actuator/health/readiness
            port: 8080
          initialDelaySeconds: 5
          periodSeconds: 5
          timeoutSeconds: 3
        resources:
          requests:
            memory: "512Mi"
            cpu: "250m"
          limits:
            memory: "1Gi"
            cpu: "500m"
        env:
        - name: SPRING_PROFILES_ACTIVE
          value: "production"
        - name: JAVA_OPTS
          value: "-Xmx512m -Xms256m -XX:+UseG1GC -XX:MaxGCPauseMillis=200"
---
apiVersion: v1
kind: Service
metadata:
  name: java-app-service
  namespace: production
spec:
  selector:
    app: java-app
  ports:
  - port: 80
    targetPort: 8080
  type: ClusterIP
```

### 6.2 健康检查配置
```yaml
# application-prod.yml
management:
  endpoints:
    web:
      exposure:
        include: health,metrics,info,prometheus
      base-path: /actuator
  endpoint:
    health:
      show-details: always
      probes:
        enabled: true
      group:
        liveness:
          include: livenessState
        readiness:
          include: readinessState
  health:
    defaults:
      enabled: true
    db:
      enabled: true
    redis:
      enabled: true
    diskspace:
      enabled: true

server:
  port: 8080
  compression:
    enabled: true
    mime-types: text/html,text/xml,text/plain,text/css,text/javascript,application/javascript,application/json
    min-response-size: 1024

spring:
  datasource:
    hikari:
      maximum-pool-size: 10
      minimum-idle: 2
      connection-timeout: 30000
      idle-timeout: 600000
      max-lifetime: 1800000
  jpa:
    open-in-view: false
    properties:
      hibernate:
        jdbc:
          batch_size: 50
        order_inserts: true
        order_updates: true
```

## 7. 性能优化

### 7.1 JVM 调优配置
```bash
#!/bin/bash
# jvm-optimization.sh

# JVM 内存配置
export JAVA_OPTS="-Xms512m -Xmx1024m -XX:MaxMetaspaceSize=256m"

# GC 配置
export JAVA_OPTS="$JAVA_OPTS -XX:+UseG1GC -XX:MaxGCPauseMillis=200"

# 性能监控
export JAVA_OPTS="$JAVA_OPTS -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=/tmp"

# 容器感知
export JAVA_OPTS="$JAVA_OPTS -XX:+UseContainerSupport -XX:MaxRAMPercentage=75.0"

# 日志配置
export JAVA_OPTS="$JAVA_OPTS -Xlog:gc*:file=/tmp/gc.log:time,uptime,level,tags:filecount=5,filesize=10m"

echo "JVM 优化配置: $JAVA_OPTS"
```

### 7.2 构建优化配置
```xml
<!-- pom.xml 构建优化 -->
<profile>
    <id>ci</id>
    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.11.0</version>
                <configuration>
                    <source>17</source>
                    <target>17</target>
                    <compilerArgs>
                        <arg>-parameters</arg>
                    </compilerArgs>
                    <fork>true</fork>
                    <meminitial>512m</meminitial>
                    <maxmem>1024m</maxmem>
                </configuration>
            </plugin>

            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-surefire-plugin</artifactId>
                <version>3.1.2</version>
                <configuration>
                    <parallel>methods</parallel>
                    <threadCount>4</threadCount>
                    <useUnlimitedThreads>false</useUnlimitedThreads>
                    <perCoreThreadCount>true</perCoreThreadCount>
                    <forkCount>2</forkCount>
                    <reuseForks>true</reuseForks>
                </configuration>
            </plugin>

            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-jar-plugin</artifactId>
                <version>3.3.0</version>
                <configuration>
                    <archive>
                        <manifest>
                            <addClasspath>true</addClasspath>
                            <mainClass>com.example.Application</mainClass>
                        </manifest>
                    </archive>
                </configuration>
            </plugin>
        </plugins>
    </build>
</profile>
```

## 8. 安全最佳实践

### 8.1 安全扫描集成
```yaml
# .github/workflows/security-scan.yml
name: Java Security Scan

on:
  schedule:
    - cron: '0 0 * * 0'
  push:
    branches: [ main ]

jobs:
  dependency-check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Java
        uses: actions/setup-java@v3
        with:
          java-version: '17'
          distribution: 'temurin'
      
      - name: Run OWASP Dependency Check
        uses: dependency-check/DependencyCheck@main
        with:
          project: 'Java Project'
          path: '.'
          format: 'HTML'
          out: 'reports'
          args: '--enableExperimental --failOnCVSS 7'
      
      - name: Upload security report
        uses: actions/upload-artifact@v3
        with:
          name: dependency-check-report
          path: reports

  snyk-scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Run Snyk to check for vulnerabilities
        uses: snyk/actions/maven@master
        env:
          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
        with:
          args: --severity-threshold=high

  trivy-scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Build Docker image
        run: docker build -t java-app:latest .
      
      - name: Run Trivy vulnerability scanner
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: 'java-app:latest'
          format: 'table'
          exit-code: '1'
          severity: 'CRITICAL,HIGH'
          ignore-unfixed: true
```

### 8.2 安全加固配置
```bash
#!/bin/bash
# security-hardening.sh

# 移除调试信息
strip -s target/*.jar 2>/dev/null || true

# 检查文件权限
find target/ -name "*.jar" -exec chmod 755 {} \;

# 检查依赖安全性
mvn org.owasp:dependency-check-maven:check -DfailOnCVSS=7

# 生成SBOM
mvn org.cyclonedx:cyclonedx-maven-plugin:makeAggregateBom -Dcyclonedx.skipAttach=true

echo "安全加固完成"
```
