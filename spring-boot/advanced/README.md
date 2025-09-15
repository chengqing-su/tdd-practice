# Spring Boot Advanced Problems

These advanced problems tackle enterprise-grade distributed systems, microservices architecture, and cloud-native development patterns. Perfect for senior developers working on complex, scalable applications.

[中文版本](#spring-boot-高级问题)

## Problems

1. **[Microservice Order System](01-microservice-order-system.md)** - Distributed transactions, saga pattern, circuit breakers, event-driven architecture

*More advanced problems will be added based on community needs and feedback.*

## What You'll Master

### Distributed Systems Architecture
- **Microservices Design** - Service decomposition and boundaries
- **Inter-Service Communication** - Synchronous and asynchronous patterns
- **Data Consistency** - Distributed transaction patterns
- **Service Discovery** - Dynamic service registration and discovery
- **Load Balancing** - Traffic distribution strategies

### Advanced Spring Cloud Features
- **Spring Cloud Gateway** - API gateway patterns and routing
- **Spring Cloud Config** - Centralized configuration management
- **Spring Cloud Sleuth** - Distributed tracing and observability
- **Spring Cloud Stream** - Event-driven microservices
- **Spring Cloud Circuit Breaker** - Fault tolerance patterns

### Event-Driven Architecture
- **Apache Kafka Integration** - Event streaming and processing
- **Saga Pattern** - Distributed transaction management
- **Event Sourcing** - Event-based state management
- **CQRS** - Command Query Responsibility Segregation
- **Event Choreography** - Decentralized coordination

### Observability & Monitoring
- **Distributed Tracing** - Request flow across services
- **Metrics Collection** - Performance and business metrics
- **Centralized Logging** - Log aggregation and analysis
- **Health Checks** - Service health monitoring
- **Alerting** - Proactive issue detection

### Advanced Testing Strategies
- **Contract Testing** - Consumer-driven contracts
- **Chaos Engineering** - Fault injection and recovery testing
- **Performance Testing** - Load and stress testing
- **End-to-End Testing** - Full system workflow validation
- **Service Virtualization** - Mock external dependencies

## Prerequisites

- Mastery of [Intermediate Problems](../intermediate/)
- Deep understanding of Spring Boot and Spring Cloud
- Experience with distributed systems concepts
- Knowledge of containerization (Docker, Kubernetes)
- Understanding of cloud platforms (AWS, Azure, GCP)

## Architecture Patterns

### Microservices Design Principles
- **Single Responsibility** - Each service has a focused purpose
- **Autonomous** - Services can be developed and deployed independently
- **Decentralized** - Avoid centralized data and governance
- **Resilient** - Handle failures gracefully
- **Observable** - Comprehensive monitoring and tracing

### Communication Patterns
- **Synchronous** - REST APIs, GraphQL
- **Asynchronous** - Message queues, event streaming
- **Request-Response** - Direct service calls
- **Event-Driven** - Publish-subscribe patterns
- **Saga** - Distributed transaction coordination

### Data Management Patterns
- **Database per Service** - Data ownership and isolation
- **Event Sourcing** - Event-based state reconstruction
- **CQRS** - Separate read and write models
- **Data Synchronization** - Eventual consistency patterns
- **Distributed Caching** - Performance optimization

## Testing Complex Systems

### Contract Testing
```java
// Consumer contract test
@SpringBootTest
@AutoConfigureStubRunner(
    stubsMode = StubRunnerProperties.StubsMode.LOCAL,
    ids = "com.example:payment-service:+:stubs"
)
class PaymentServiceContractTest {

    @Autowired
    private PaymentServiceClient paymentClient;

    @Test
    void shouldProcessPayment() {
        // Given
        PaymentRequest request = new PaymentRequest("order-123", new BigDecimal("100.00"));

        // When
        PaymentResponse response = paymentClient.processPayment(request);

        // Then
        assertThat(response.getStatus()).isEqualTo("SUCCESS");
        assertThat(response.getTransactionId()).isNotNull();
    }
}
```

### Distributed System Integration Tests
```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@Testcontainers
class DistributedSystemIntegrationTest {

    @Container
    static GenericContainer<?> kafka = new KafkaContainer(DockerImageName.parse("confluentinc/cp-kafka:latest"));

    @Container
    static Network network = Network.newNetwork();

    @Container
    static GenericContainer<?> orderService = new GenericContainer<>("order-service:latest")
            .withNetwork(network)
            .withNetworkAliases("order-service");

    @Container
    static GenericContainer<?> paymentService = new GenericContainer<>("payment-service:latest")
            .withNetwork(network)
            .withNetworkAliases("payment-service");

    @Test
    void shouldProcessOrderEndToEnd() {
        // Test complete order flow across multiple services
    }
}
```

### Chaos Engineering
```java
@Component
public class ChaosEngineer {

    public void introduceLatency(String serviceName, Duration latency) {
        // Introduce network latency
    }

    public void simulateServiceFailure(String serviceName, Duration duration) {
        // Simulate service downtime
    }

    public void corruptMessages(String topic, double percentage) {
        // Introduce message corruption
    }
}

@Test
void shouldHandleServiceFailureGracefully() {
    // Given
    chaosEngineer.simulateServiceFailure("payment-service", Duration.ofMinutes(2));

    // When
    String orderId = orderService.createOrder(orderRequest);

    // Then - Should handle gracefully with circuit breaker
    await().atMost(Duration.ofMinutes(5)).untilAsserted(() -> {
        Order order = orderService.getOrder(orderId);
        assertThat(order.getStatus()).isIn(OrderStatus.PENDING, OrderStatus.FAILED);
    });
}
```

## Performance & Scalability

### Load Testing
```java
@Test
void shouldHandleHighLoad() {
    int numberOfThreads = 100;
    int numberOfRequestsPerThread = 1000;

    ExecutorService executor = Executors.newFixedThreadPool(numberOfThreads);
    CountDownLatch latch = new CountDownLatch(numberOfThreads);

    List<CompletableFuture<Void>> futures = IntStream.range(0, numberOfThreads)
            .mapToObj(i -> CompletableFuture.runAsync(() -> {
                try {
                    for (int j = 0; j < numberOfRequestsPerThread; j++) {
                        orderService.createOrder(generateOrderRequest());
                    }
                } finally {
                    latch.countDown();
                }
            }, executor))
            .collect(Collectors.toList());

    // Wait for completion and assert results
    assertThat(latch.await(10, TimeUnit.MINUTES)).isTrue();
    futures.forEach(CompletableFuture::join);

    // Verify system performance under load
    assertThat(metricsCollector.getAverageResponseTime()).isLessThan(Duration.ofMillis(500));
    assertThat(metricsCollector.getErrorRate()).isLessThan(0.01); // Less than 1% error rate
}
```

### Memory and Resource Testing
```java
@Test
void shouldNotLeakMemory() {
    // Given - Baseline memory measurement
    long initialMemory = getUsedMemory();

    // When - Process many orders
    for (int i = 0; i < 10000; i++) {
        orderService.createOrder(generateOrderRequest());
    }

    // Force garbage collection
    System.gc();
    Thread.sleep(1000);

    // Then - Memory should not grow significantly
    long finalMemory = getUsedMemory();
    long memoryIncrease = finalMemory - initialMemory;
    assertThat(memoryIncrease).isLessThan(100 * 1024 * 1024); // Less than 100MB increase
}
```

## Security in Distributed Systems

### Service-to-Service Authentication
```java
@Configuration
@EnableGlobalMethodSecurity(prePostEnabled = true)
public class SecurityConfig {

    @Bean
    public ReactiveJwtDecoder jwtDecoder() {
        NimbusReactiveJwtDecoder decoder = NimbusReactiveJwtDecoder
                .withJwkSetUri("https://auth-service/.well-known/jwks.json")
                .build();

        decoder.setJwtValidator(jwtValidator());
        return decoder;
    }

    @Bean
    public Converter<Jwt, AbstractAuthenticationToken> jwtAuthenticationConverter() {
        return new ReactiveJwtAuthenticationConverter();
    }
}

// Service-to-service call with JWT
@Component
public class SecureServiceClient {

    private final WebClient webClient;

    public SecureServiceClient(WebClient.Builder builder) {
        this.webClient = builder
            .filter(oauth2Credentials())
            .build();
    }

    private ExchangeFilterFunction oauth2Credentials() {
        return ExchangeFilterFunction.ofRequestProcessor(clientRequest -> {
            return getServiceToken()
                .map(token -> ClientRequest.from(clientRequest)
                    .header(HttpHeaders.AUTHORIZATION, "Bearer " + token)
                    .build());
        });
    }
}
```

### API Rate Limiting
```java
@Component
public class RateLimitingFilter implements WebFilter {

    private final RedisTemplate<String, String> redisTemplate;
    private final RateLimiter rateLimiter;

    @Override
    public Mono<Void> filter(ServerWebExchange exchange, WebFilterChain chain) {
        String clientId = extractClientId(exchange.getRequest());

        return Mono.fromCallable(() -> rateLimiter.tryAcquire(clientId))
            .flatMap(allowed -> {
                if (allowed) {
                    return chain.filter(exchange);
                } else {
                    return handleRateLimit(exchange);
                }
            });
    }
}
```

## Deployment & DevOps

### Container Health Checks
```java
@RestController
public class HealthController {

    private final OrderService orderService;
    private final KafkaHealthIndicator kafkaHealth;
    private final DatabaseHealthIndicator databaseHealth;

    @GetMapping("/health")
    public ResponseEntity<Map<String, Object>> health() {
        Map<String, Object> health = new HashMap<>();

        health.put("status", "UP");
        health.put("database", databaseHealth.check());
        health.put("kafka", kafkaHealth.check());
        health.put("dependencies", checkDependencies());

        return ResponseEntity.ok(health);
    }

    @GetMapping("/health/readiness")
    public ResponseEntity<String> readiness() {
        if (orderService.isReady()) {
            return ResponseEntity.ok("READY");
        } else {
            return ResponseEntity.status(503).body("NOT_READY");
        }
    }
}
```

### Kubernetes Deployment
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: order-service
spec:
  replicas: 3
  selector:
    matchLabels:
      app: order-service
  template:
    metadata:
      labels:
        app: order-service
    spec:
      containers:
      - name: order-service
        image: order-service:latest
        ports:
        - containerPort: 8080
        env:
        - name: SPRING_PROFILES_ACTIVE
          value: "kubernetes"
        livenessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 60
          periodSeconds: 30
        readinessProbe:
          httpGet:
            path: /health/readiness
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
        resources:
          requests:
            memory: "512Mi"
            cpu: "500m"
          limits:
            memory: "1Gi"
            cpu: "1000m"
```

## Best Practices

### Error Handling
- Implement circuit breakers for external dependencies
- Use bulkhead patterns to isolate failures
- Implement proper timeout strategies
- Log errors with correlation IDs for tracing

### Performance Optimization
- Use connection pooling for database and HTTP clients
- Implement caching at multiple levels
- Use asynchronous processing for long-running operations
- Monitor and optimize garbage collection

### Security
- Implement zero-trust security model
- Use mutual TLS for service-to-service communication
- Implement proper secret management
- Regular security audits and dependency updates

### Monitoring & Observability
- Implement distributed tracing across all services
- Use structured logging with correlation IDs
- Monitor business metrics, not just technical ones
- Implement proper alerting with reduced noise

## Career Development

Mastering these advanced concepts will prepare you for:
- **Senior Software Architect** roles
- **Platform Engineering** positions
- **DevOps/SRE** responsibilities
- **Technical Leadership** opportunities
- **Consulting** and architecture design

---

# Spring Boot 高级问题

这些高级问题处理企业级分布式系统、微服务架构和云原生开发模式。非常适合从事复杂、可扩展应用程序的高级开发者。

## 问题列表

1. **[微服务订单系统](01-microservice-order-system.md)** - 分布式事务、saga模式、熔断器、事件驱动架构

*更多高级问题将根据社区需求和反馈添加。*

## 你将掌握什么

### 分布式系统架构
- **微服务设计** - 服务分解和边界
- **服务间通信** - 同步和异步模式
- **数据一致性** - 分布式事务模式
- **服务发现** - 动态服务注册和发现
- **负载均衡** - 流量分发策略

### 高级Spring Cloud功能
- **Spring Cloud Gateway** - API网关模式和路由
- **Spring Cloud Config** - 集中配置管理
- **Spring Cloud Sleuth** - 分布式追踪和可观测性
- **Spring Cloud Stream** - 事件驱动微服务
- **Spring Cloud Circuit Breaker** - 容错模式

## 最佳实践

### 错误处理
- 为外部依赖实现熔断器
- 使用舱壁模式隔离故障
- 实现适当的超时策略
- 用关联ID记录错误以便追踪

### 性能优化
- 为数据库和HTTP客户端使用连接池
- 在多个级别实现缓存
- 对长时间运行的操作使用异步处理
- 监控和优化垃圾收集

### 安全
- 实施零信任安全模型
- 服务间通信使用双向TLS
- 实施适当的密钥管理
- 定期安全审计和依赖更新

### 监控和可观测性
- 在所有服务中实现分布式追踪
- 使用结构化日志记录和关联ID
- 监控业务指标，不仅仅是技术指标
- 实施适当的低噪音告警

## 职业发展

掌握这些高级概念将为你准备：
- **高级软件架构师**角色
- **平台工程**职位
- **DevOps/SRE**职责
- **技术领导**机会
- **咨询**和架构设计