# Java 微服务技术栈

## 概述

Java生态系统为微服务架构提供了最成熟和全面的技术选择。从轻量级框架到全功能平台，Java在微服务领域有着丰富的解决方案。

## 核心框架选择

### 1. Spring Boot + Spring Cloud
**最流行的Java微服务框架组合**

```xml
<!-- Spring Boot Starter -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>

<!-- Spring Cloud Netflix Eureka -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

**核心组件：**
- **Spring Boot**: 快速应用开发
- **Spring Cloud**: 微服务治理套件
- **Spring Cloud Netflix**: Netflix OSS集成

### 2. Micronaut
**编译时依赖注入，启动快，内存占用低**

```xml
<dependency>
    <groupId>io.micronaut</groupId>
    <artifactId>micronaut-http-server-netty</artifactId>
</dependency>
```

**优势：**
- 超快启动时间（毫秒级）
- 低内存消耗
- 编译时AOP

### 3. Quarkus
**Kubernetes原生Java框架**

```xml
<dependency>
    <groupId>io.quarkus</groupId>
    <artifactId>quarkus-resteasy</artifactId>
</dependency>
```

**特点：**
- 容器优先设计
- 极低的内存占用
-  GraalVM原生镜像支持

### 4. Helidon
**Oracle开发的轻量级框架**

```xml
<dependency>
    <groupId>io.helidon</groupId>
    <artifactId>helidon-microprofile</artifactId>
</dependency>
```

**优势：**
- 两种编程模型：SE和MP
- 简单的API设计
- 良好的性能

## 服务治理组件

### 服务发现与注册
```java
// Eureka客户端配置
@SpringBootApplication
@EnableEurekaClient
public class UserServiceApplication {
    public static void main(String[] args) {
        SpringApplication.run(UserServiceApplication.class, args);
    }
}
```

**注册中心选项：**
- **Eureka**: Netflix服务发现
- **Consul**: 多数据中心支持
- **Zookeeper**: Apache分布式协调
- **Nacos**: 阿里巴巴服务发现和配置管理

### API网关
```java
// Spring Cloud Gateway配置
@Bean
public RouteLocator customRouteLocator(RouteLocatorBuilder builder) {
    return builder.routes()
        .route("user_route", r -> r.path("/api/users/**")
            .uri("lb://user-service"))
        .build();
}
```

**网关方案：**
- **Spring Cloud Gateway**: 响应式API网关
- **Netflix Zuul**: 传统网关（已进入维护）
- **Kong**: 高性能API网关
- **Apache APISIX**: 云原生API网关

### 配置管理
```yaml
# application.yml
spring:
  cloud:
    config:
      uri: http://config-server:8888
      label: main
```

**配置中心：**
- **Spring Cloud Config**: Git后端配置管理
- **Consul**: Key-Value配置存储
- **Nacos**: 动态配置管理
- **Apollo**: 携程开源配置中心

## 通信机制

### RESTful API
```java
// Spring WebFlux响应式API
@RestController
@RequestMapping("/api/users")
public class UserController {
    
    @GetMapping("/{id}")
    public Mono<User> getUser(@PathVariable Long id) {
        return userService.findById(id);
    }
}
```

### RPC调用
```java
// gRPC服务定义
service UserService {
    rpc GetUser (UserRequest) returns (UserResponse);
}

// Feign客户端
@FeignClient(name = "order-service")
public interface OrderClient {
    @GetMapping("/orders/user/{userId}")
    List<Order> getUserOrders(@PathVariable Long userId);
}
```

**RPC框架：**
- **gRPC**: 高性能RPC框架
- **Apache Thrift**: Facebook跨语言RPC
- **Dubbo**: 阿里巴巴RPC框架

### 消息队列
```java
// Spring Kafka示例
@KafkaListener(topics = "order-events")
public void handleOrderEvent(OrderEvent event) {
    orderService.processOrder(event);
}

@Autowired
private KafkaTemplate<String, Object> kafkaTemplate;

public void sendOrderEvent(OrderEvent event) {
    kafkaTemplate.send("order-events", event);
}
```

**消息中间件：**
- **Kafka**: 高吞吐量消息系统
- **RabbitMQ**: AMQP消息代理
- **RocketMQ**: 阿里巴巴消息队列
- **ActiveMQ**: JMS消息队列

## 容错与 resilience

### 熔断器模式
```java
// Resilience4j熔断器
@CircuitBreaker(name = "userService", fallbackMethod = "fallback")
public User getUser(Long id) {
    return userClient.getUser(id);
}

public User fallback(Long id, Exception e) {
    return new User(id, "Fallback User");
}
```

**容错库：**
- **Resilience4j**: 轻量级容错库
- **Hystrix**: Netflix容错库（已进入维护）
- **Spring Retry**: 重试机制

### 限流与降级
```java
// Resilience4j限流
@RateLimiter(name = "userService")
public List<User> getUsers() {
    return userClient.getUsers();
}
```

## 数据管理

### 数据库访问
```java
// Spring Data JPA
public interface UserRepository extends JpaRepository<User, Long> {
    List<User> findByStatus(String status);
    
    @Query("SELECT u FROM User u WHERE u.age > :age")
    List<User> findUsersOlderThan(@Param("age") int age);
}
```

**数据访问技术：**
- **Spring Data JPA**: ORM抽象
- **MyBatis**: SQL映射框架
- **JOOQ**: 类型安全SQL构建
- **R2DBC**: 响应式数据库访问

### 分布式事务
```java
// Seata分布式事务
@GlobalTransactional
public void createOrder(Order order) {
    orderRepository.save(order);
    inventoryService.deductStock(order.getProductId(), order.getQuantity());
    // 如果库存扣减失败，订单创建会回滚
}
```

**事务方案：**
- **Seata**: 阿里巴巴分布式事务
- **Spring Cloud Saga**:  Saga模式实现
- **LCN**: 锁补偿事务

### 缓存策略
```java
// Spring Cache抽象
@Cacheable(value = "users", key = "#id")
public User getUser(Long id) {
    return userRepository.findById(id).orElseThrow();
}
```

**缓存方案：**
- **Redis**: 内存数据存储
- **Ehcache**: Java进程内缓存
- **Caffeine**: 高性能缓存库
- **Memcached**: 分布式内存缓存

## 安全认证

### OAuth2 + JWT
```java
// Spring Security OAuth2
@EnableWebSecurity
@EnableGlobalMethodSecurity(prePostEnabled = true)
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.oauth2ResourceServer().jwt();
    }
}
```

**安全方案：**
- **Spring Security**: 认证授权框架
- **Keycloak**: 开源身份管理
- **Auth0**: 商业身份平台

## 监控与可观测性

### 指标收集
```java
// Micrometer指标
MeterRegistry registry = new PrometheusMeterRegistry();
Counter counter = registry.counter("http.requests", "endpoint", "/api/users");
counter.increment();
```

**监控栈：**
- **Micrometer**: 指标门面
- **Prometheus**: 指标收集
- **Grafana**: 数据可视化

### 分布式追踪
```java
// OpenTelemetry追踪
Tracer tracer = OpenTelemetry.getTracer("user-service");
Span span = tracer.spanBuilder("processOrder").startSpan();
try (Scope scope = span.makeCurrent()) {
    // 业务逻辑
} finally {
    span.end();
}
```

**追踪系统：**
- **Jaeger**
- **Zipkin**
- **SkyWalking**

### 日志管理
```java
// SLF4J + Logback
private static final Logger logger = LoggerFactory.getLogger(UserService.class);

logger.info("User created: {}", userId);
logger.error("Failed to process order", exception);
```

**日志方案：**
- **ELK Stack**: Elasticsearch, Logstash, Kibana
- **Loki**: Grafana日志聚合
- **Splunk**: 商业日志管理

## 容器化与部署

### Docker化
```dockerfile
FROM openjdk:17-jre-slim

WORKDIR /app
COPY target/user-service.jar app.jar

EXPOSE 8080

ENTRYPOINT ["java", "-jar", "app.jar"]
```

### Kubernetes部署
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: user-service
spec:
  replicas: 3
  template:
    spec:
      containers:
      - name: user-service
        image: user-service:1.0.0
        ports:
        - containerPort: 8080
        env:
        - name: SPRING_PROFILES_ACTIVE
          value: "prod"
        resources:
          requests:
            memory: "512Mi"
            cpu: "250m"
          limits:
            memory: "1Gi"
            cpu: "500m"
        livenessProbe:
          httpGet:
            path: /actuator/health
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
```

## 测试策略

### 单元测试
```java
// JUnit 5 + Mockito
@Test
void shouldCreateUser() {
    User user = new User("test@example.com");
    when(userRepository.save(any(User.class))).thenReturn(user);
    
    User created = userService.createUser("test@example.com");
    
    assertNotNull(created);
    assertEquals("test@example.com", created.getEmail());
}
```

### 集成测试
```java
// Spring Boot Test
@SpringBootTest
@AutoConfigureMockMvc
class UserControllerIntegrationTest {
    
    @Autowired
    private MockMvc mockMvc;
    
    @Test
    void shouldReturnUser() throws Exception {
        mockMvc.perform(get("/api/users/1"))
               .andExpect(status().isOk())
               .andExpect(jsonPath("$.name").value("John Doe"));
    }
}
```

### 契约测试
```java
// Spring Cloud Contract
Contract.make {
    request {
        method GET()
        url "/api/users/1"
    }
    response {
        status OK()
        body("""
            {
                "id": 1,
                "name": "John Doe"
            }
        """)
        headers {
            contentType(applicationJson())
        }
    }
}
```

## 开发工具链

### 构建工具
```xml
<!-- Maven示例 -->
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```

**构建工具：**
- **Maven**: 传统构建工具
- **Gradle**: 灵活构建工具
- **Bazel**: 谷歌构建工具

### 开发工具
```markdown
- **IntelliJ IDEA**: 最佳Java IDE
- **Visual Studio Code**: 轻量级编辑器
- **Eclipse**: 传统IDE
```

### CI/CD流水线
```yaml
# GitHub Actions示例
name: Java CI

on: [push]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v2
    - name: Set up JDK 17
      uses: actions/setup-java@v2
      with:
        java-version: '17'
        distribution: 'temurin'
    - name: Build with Maven
      run: mvn clean package -DskipTests
    - name: Run tests
      run: mvn test
    - name: Build Docker image
      run: docker build -t user-service:${{ github.sha }} .
```

## 性能优化

### JVM调优
```bash
# JVM参数示例
java -Xms512m -Xmx1g \
     -XX:+UseG1GC \
     -XX:MaxGCPauseMillis=200 \
     -jar app.jar
```

### 响应式编程
```java
// WebFlux响应式API
@GetMapping("/users")
public Flux<User> getUsers() {
    return userService.findAllUsers();
}

@PostMapping("/users")
public Mono<User> createUser(@RequestBody User user) {
    return userService.saveUser(user);
}
```

## 最佳实践

### 12-Factor应用原则
```markdown
1. 代码库：一个代码库，多个部署
2. 依赖：显式声明依赖关系
3. 配置：在环境中存储配置
4. 后端服务：视为附加资源
5. 构建、发布、运行：严格分离构建和运行
6. 进程：以一个或多个无状态进程运行
7. 端口绑定：通过端口绑定提供服务
8. 并发：通过进程模型进行扩展
9. 易处理：快速启动和优雅终止
10. 开发环境与线上环境等价：尽可能保持相似
11. 日志：把日志当作事件流
12. 管理进程：一次性管理任务作为单独进程运行
```

### 微服务设计模式
```markdown
- **API网关模式**: 统一入口点
- **熔断器模式**: 防止级联故障
- **服务发现模式**: 动态服务定位
- **配置服务器模式**: 集中配置管理
- **分布式追踪模式**: 请求链路追踪
```

## 总结

> 重要：Java微服务生态非常丰富，选择合适的技术栈需要根据团队技能、项目规模、性能要求和运维能力综合考虑。

**推荐技术栈组合：**
- **传统企业**: Spring Boot + Spring Cloud + Eureka + Zuul
- **云原生**: Spring Boot + Kubernetes + Istio + gRPC
- **高性能**: Quarkus/Micronaut + GraalVM + Redis
- **全响应式**: Spring WebFlux + R2DBC + Reactor

***
*相关阅读：./spring-cloud-practice.md | ./java-kubernetes.md | ./microservice-monitoring.md*