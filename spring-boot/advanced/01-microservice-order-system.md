# Microservice Order System

## Problem Description

Build a distributed order processing system using multiple Spring Boot microservices with asynchronous communication, distributed transactions, circuit breakers, and comprehensive observability.

## Architecture Overview

This system consists of multiple microservices:
- **Order Service** - Manages order lifecycle
- **Payment Service** - Processes payments
- **Inventory Service** - Manages stock levels
- **Notification Service** - Sends notifications
- **API Gateway** - Routes requests and handles cross-cutting concerns

## Requirements

### Microservices Communication
- Asynchronous messaging with Apache Kafka
- RESTful APIs for synchronous communication
- Service discovery with Spring Cloud Netflix Eureka
- Load balancing with Spring Cloud LoadBalancer
- Circuit breaker pattern with Resilience4j

### Distributed Transaction Management
- Saga pattern implementation
- Compensating transactions
- Event-driven choreography
- Distributed tracing with Sleuth/Zipkin

### Observability & Monitoring
- Distributed tracing across services
- Metrics collection with Micrometer
- Health checks and circuit breaker monitoring
- Centralized logging

### Testing Strategy
- Contract testing with Spring Cloud Contract
- Integration testing across services
- Chaos engineering principles
- Performance testing under load

## Project Setup

### Dependencies (Order Service)
```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.kafka</groupId>
        <artifactId>spring-kafka</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-circuitbreaker-resilience4j</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-sleuth</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-sleuth-zipkin</artifactId>
    </dependency>
    <dependency>
        <groupId>io.micrometer</groupId>
        <artifactId>micrometer-registry-prometheus</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>org.testcontainers</groupId>
        <artifactId>kafka</artifactId>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-contract-wiremock</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>
```

## Domain Model

### Order Service Entities
```java
@Entity
@Table(name = "orders")
public class Order {
    @Id
    private String orderId;

    @Column(nullable = false)
    private String customerId;

    @OneToMany(mappedBy = "order", cascade = CascadeType.ALL, fetch = FetchType.LAZY)
    private List<OrderItem> items = new ArrayList<>();

    @Enumerated(EnumType.STRING)
    private OrderStatus status = OrderStatus.PENDING;

    @Column(precision = 10, scale = 2)
    private BigDecimal totalAmount;

    @CreatedDate
    private LocalDateTime createdAt;

    @LastModifiedDate
    private LocalDateTime updatedAt;

    // constructors, getters, setters...
}

@Entity
@Table(name = "order_items")
public class OrderItem {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "order_id")
    private Order order;

    private String productId;
    private Integer quantity;

    @Column(precision = 10, scale = 2)
    private BigDecimal unitPrice;

    // constructors, getters, setters...
}

enum OrderStatus {
    PENDING, CONFIRMED, PAYMENT_PENDING, PAID, SHIPPED, DELIVERED, CANCELLED
}
```

### Saga State Machine
```java
@Entity
@Table(name = "order_saga")
public class OrderSaga {
    @Id
    private String orderId;

    @Enumerated(EnumType.STRING)
    private SagaStatus status = SagaStatus.STARTED;

    @ElementCollection(fetch = FetchType.EAGER)
    @Enumerated(EnumType.STRING)
    private Set<SagaStep> completedSteps = new HashSet<>();

    @ElementCollection(fetch = FetchType.EAGER)
    @Enumerated(EnumType.STRING)
    private Set<SagaStep> compensatedSteps = new HashSet<>();

    private LocalDateTime createdAt;
    private LocalDateTime updatedAt;

    // constructors, getters, setters...
}

enum SagaStatus {
    STARTED, COMPLETED, FAILED, COMPENSATING, COMPENSATED
}

enum SagaStep {
    INVENTORY_RESERVED, PAYMENT_PROCESSED, ORDER_CONFIRMED, NOTIFICATION_SENT
}
```

## Saga Pattern Implementation

### Order Saga Service
```java
@Service
@Transactional
public class OrderSagaService {

    private final OrderSagaRepository sagaRepository;
    private final OrderRepository orderRepository;
    private final KafkaTemplate<String, Object> kafkaTemplate;
    private final MeterRegistry meterRegistry;

    public OrderSagaService(OrderSagaRepository sagaRepository,
                           OrderRepository orderRepository,
                           KafkaTemplate<String, Object> kafkaTemplate,
                           MeterRegistry meterRegistry) {
        this.sagaRepository = sagaRepository;
        this.orderRepository = orderRepository;
        this.kafkaTemplate = kafkaTemplate;
        this.meterRegistry = meterRegistry;
    }

    public void startOrderSaga(String orderId) {
        OrderSaga saga = new OrderSaga();
        saga.setOrderId(orderId);
        saga.setStatus(SagaStatus.STARTED);
        saga.setCreatedAt(LocalDateTime.now());
        sagaRepository.save(saga);

        // Start the saga by reserving inventory
        reserveInventory(orderId);

        meterRegistry.counter("saga.started", "type", "order").increment();
    }

    @KafkaListener(topics = "inventory-reserved", groupId = "order-service")
    public void handleInventoryReserved(InventoryReservedEvent event) {
        OrderSaga saga = getSaga(event.getOrderId());

        saga.getCompletedSteps().add(SagaStep.INVENTORY_RESERVED);
        saga.setUpdatedAt(LocalDateTime.now());
        sagaRepository.save(saga);

        // Next step: process payment
        processPayment(event.getOrderId(), event.getTotalAmount());
    }

    @KafkaListener(topics = "payment-processed", groupId = "order-service")
    public void handlePaymentProcessed(PaymentProcessedEvent event) {
        OrderSaga saga = getSaga(event.getOrderId());

        saga.getCompletedSteps().add(SagaStep.PAYMENT_PROCESSED);
        saga.setUpdatedAt(LocalDateTime.now());
        sagaRepository.save(saga);

        // Next step: confirm order
        confirmOrder(event.getOrderId());
    }

    @KafkaListener(topics = "payment-failed", groupId = "order-service")
    public void handlePaymentFailed(PaymentFailedEvent event) {
        OrderSaga saga = getSaga(event.getOrderId());
        compensateSaga(saga);
    }

    @KafkaListener(topics = "inventory-reservation-failed", groupId = "order-service")
    public void handleInventoryReservationFailed(InventoryReservationFailedEvent event) {
        OrderSaga saga = getSaga(event.getOrderId());
        failSaga(saga);
    }

    private void confirmOrder(String orderId) {
        Order order = orderRepository.findById(orderId)
            .orElseThrow(() -> new OrderNotFoundException("Order not found: " + orderId));

        order.setStatus(OrderStatus.CONFIRMED);
        orderRepository.save(order);

        // Publish order confirmed event
        OrderConfirmedEvent event = new OrderConfirmedEvent(orderId, order.getCustomerId());
        kafkaTemplate.send("order-confirmed", event);

        // Update saga
        OrderSaga saga = getSaga(orderId);
        saga.getCompletedSteps().add(SagaStep.ORDER_CONFIRMED);
        saga.setStatus(SagaStatus.COMPLETED);
        saga.setUpdatedAt(LocalDateTime.now());
        sagaRepository.save(saga);

        meterRegistry.counter("saga.completed", "type", "order").increment();
    }

    private void compensateSaga(OrderSaga saga) {
        saga.setStatus(SagaStatus.COMPENSATING);
        saga.setUpdatedAt(LocalDateTime.now());
        sagaRepository.save(saga);

        // Compensate in reverse order
        if (saga.getCompletedSteps().contains(SagaStep.PAYMENT_PROCESSED)) {
            refundPayment(saga.getOrderId());
        }

        if (saga.getCompletedSteps().contains(SagaStep.INVENTORY_RESERVED)) {
            releaseInventory(saga.getOrderId());
        }

        meterRegistry.counter("saga.compensated", "type", "order").increment();
    }

    private void reserveInventory(String orderId) {
        Order order = orderRepository.findById(orderId)
            .orElseThrow(() -> new OrderNotFoundException("Order not found: " + orderId));

        InventoryReservationRequest request = new InventoryReservationRequest();
        request.setOrderId(orderId);
        request.setItems(order.getItems().stream()
            .map(item -> new ReservationItem(item.getProductId(), item.getQuantity()))
            .collect(Collectors.toList()));

        kafkaTemplate.send("reserve-inventory", request);
    }

    private void processPayment(String orderId, BigDecimal amount) {
        Order order = orderRepository.findById(orderId)
            .orElseThrow(() -> new OrderNotFoundException("Order not found: " + orderId));

        PaymentRequest request = new PaymentRequest();
        request.setOrderId(orderId);
        request.setCustomerId(order.getCustomerId());
        request.setAmount(amount);

        kafkaTemplate.send("process-payment", request);
    }

    private void refundPayment(String orderId) {
        RefundRequest request = new RefundRequest(orderId);
        kafkaTemplate.send("refund-payment", request);
    }

    private void releaseInventory(String orderId) {
        InventoryReleaseRequest request = new InventoryReleaseRequest(orderId);
        kafkaTemplate.send("release-inventory", request);
    }

    private OrderSaga getSaga(String orderId) {
        return sagaRepository.findById(orderId)
            .orElseThrow(() -> new SagaNotFoundException("Saga not found for order: " + orderId));
    }

    private void failSaga(OrderSaga saga) {
        saga.setStatus(SagaStatus.FAILED);
        saga.setUpdatedAt(LocalDateTime.now());
        sagaRepository.save(saga);

        // Update order status
        Order order = orderRepository.findById(saga.getOrderId())
            .orElseThrow(() -> new OrderNotFoundException("Order not found: " + saga.getOrderId()));
        order.setStatus(OrderStatus.CANCELLED);
        orderRepository.save(order);

        meterRegistry.counter("saga.failed", "type", "order").increment();
    }
}
```

## Circuit Breaker Integration

### Payment Service Client
```java
@Component
public class PaymentServiceClient {

    private final WebClient webClient;
    private final CircuitBreaker circuitBreaker;

    public PaymentServiceClient(WebClient.Builder webClientBuilder,
                               CircuitBreakerRegistry circuitBreakerRegistry) {
        this.webClient = webClientBuilder
            .baseUrl("http://payment-service")
            .build();
        this.circuitBreaker = circuitBreakerRegistry.circuitBreaker("payment-service");
    }

    public PaymentResponse processPayment(PaymentRequest request) {
        return circuitBreaker.executeSupplier(() ->
            webClient.post()
                .uri("/api/payments")
                .body(Mono.just(request), PaymentRequest.class)
                .retrieve()
                .onStatus(HttpStatus::is4xxClientError, response ->
                    Mono.error(new PaymentValidationException("Invalid payment request")))
                .onStatus(HttpStatus::is5xxServerError, response ->
                    Mono.error(new PaymentServiceException("Payment service error")))
                .bodyToMono(PaymentResponse.class)
                .block(Duration.ofSeconds(10))
        );
    }
}

@Configuration
public class CircuitBreakerConfig {

    @Bean
    public CircuitBreakerRegistry circuitBreakerRegistry() {
        return CircuitBreakerRegistry.of(
            CircuitBreakerConfig.custom()
                .failureRateThreshold(50)
                .waitDurationInOpenState(Duration.ofSeconds(30))
                .slidingWindowSize(10)
                .minimumNumberOfCalls(5)
                .build()
        );
    }
}
```

## Advanced Testing Patterns

### Contract Testing
```java
// Contract definition (payment-service)
@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.MOCK)
@AutoConfigureStubRunner(
    stubsMode = StubRunnerProperties.StubsMode.LOCAL,
    ids = "com.example:payment-service:+:stubs:8080"
)
public class PaymentContractTest {

    @Test
    public void shouldProcessPaymentSuccessfully() {
        // Contract test implementation
    }
}

// Contract stub
contracts {
    contract {
        description "should process payment successfully"
        request {
            method POST()
            url "/api/payments"
            body(
                customerId: "customer-123",
                amount: 100.00,
                orderId: "order-456"
            )
            headers {
                contentType(applicationJson())
            }
        }
        response {
            status OK()
            body(
                paymentId: "payment-789",
                status: "SUCCESS",
                orderId: "order-456"
            )
            headers {
                contentType(applicationJson())
            }
        }
    }
}
```

### Integration Testing with TestContainers
```java
@SpringBootTest
@Testcontainers
class OrderSagaIntegrationTest {

    @Container
    static KafkaContainer kafka = new KafkaContainer(DockerImageName.parse("confluentinc/cp-kafka:latest"));

    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:13");

    @DynamicPropertySource
    static void configureProperties(DynamicPropertyRegistry registry) {
        registry.add("spring.kafka.bootstrap-servers", kafka::getBootstrapServers);
        registry.add("spring.datasource.url", postgres::getJdbcUrl);
        registry.add("spring.datasource.username", postgres::getUsername);
        registry.add("spring.datasource.password", postgres::getPassword);
    }

    @Autowired
    private OrderSagaService orderSagaService;

    @Autowired
    private OrderRepository orderRepository;

    @Autowired
    private OrderSagaRepository sagaRepository;

    @Test
    void shouldCompleteOrderSagaSuccessfully() {
        // Given
        Order order = createTestOrder();
        orderRepository.save(order);

        // When
        orderSagaService.startOrderSaga(order.getOrderId());

        // Simulate successful inventory reservation
        InventoryReservedEvent inventoryEvent = new InventoryReservedEvent(
            order.getOrderId(), order.getTotalAmount());
        orderSagaService.handleInventoryReserved(inventoryEvent);

        // Simulate successful payment processing
        PaymentProcessedEvent paymentEvent = new PaymentProcessedEvent(
            order.getOrderId(), "payment-123");
        orderSagaService.handlePaymentProcessed(paymentEvent);

        // Then
        await().atMost(Duration.ofSeconds(10)).untilAsserted(() -> {
            OrderSaga saga = sagaRepository.findById(order.getOrderId()).orElseThrow();
            assertThat(saga.getStatus()).isEqualTo(SagaStatus.COMPLETED);
            assertThat(saga.getCompletedSteps()).containsAll(Arrays.asList(
                SagaStep.INVENTORY_RESERVED,
                SagaStep.PAYMENT_PROCESSED,
                SagaStep.ORDER_CONFIRMED
            ));

            Order updatedOrder = orderRepository.findById(order.getOrderId()).orElseThrow();
            assertThat(updatedOrder.getStatus()).isEqualTo(OrderStatus.CONFIRMED);
        });
    }

    @Test
    void shouldCompensateOnPaymentFailure() {
        // Given
        Order order = createTestOrder();
        orderRepository.save(order);

        orderSagaService.startOrderSaga(order.getOrderId());

        // Simulate successful inventory reservation
        InventoryReservedEvent inventoryEvent = new InventoryReservedEvent(
            order.getOrderId(), order.getTotalAmount());
        orderSagaService.handleInventoryReserved(inventoryEvent);

        // When - Simulate payment failure
        PaymentFailedEvent paymentFailedEvent = new PaymentFailedEvent(
            order.getOrderId(), "Insufficient funds");
        orderSagaService.handlePaymentFailed(paymentFailedEvent);

        // Then
        await().atMost(Duration.ofSeconds(10)).untilAsserted(() -> {
            OrderSaga saga = sagaRepository.findById(order.getOrderId()).orElseThrow();
            assertThat(saga.getStatus()).isEqualTo(SagaStatus.COMPENSATING);

            Order updatedOrder = orderRepository.findById(order.getOrderId()).orElseThrow();
            assertThat(updatedOrder.getStatus()).isEqualTo(OrderStatus.CANCELLED);
        });
    }

    private Order createTestOrder() {
        Order order = new Order();
        order.setOrderId("order-" + UUID.randomUUID());
        order.setCustomerId("customer-123");
        order.setTotalAmount(new BigDecimal("100.00"));

        OrderItem item = new OrderItem();
        item.setProductId("product-456");
        item.setQuantity(2);
        item.setUnitPrice(new BigDecimal("50.00"));
        item.setOrder(order);

        order.getItems().add(item);
        return order;
    }
}
```

### Chaos Engineering Tests
```java
@SpringBootTest
@Testcontainers
class ChaosEngineeringTest {

    @Container
    static ToxiproxyContainer toxiproxy = new ToxiproxyContainer("ghcr.io/shopify/toxiproxy:2.4.0");

    @Container
    static KafkaContainer kafka = new KafkaContainer(DockerImageName.parse("confluentinc/cp-kafka:latest"));

    @Test
    void shouldHandleNetworkLatency() throws IOException {
        // Given - Add network latency
        ToxiproxyClient toxiproxyClient = new ToxiproxyClient(toxiproxy.getHost(), toxiproxy.getControlPort());
        Proxy proxy = toxiproxyClient.createProxy("kafka", "0.0.0.0:8666", "kafka:9092");
        proxy.toxics().latency("latency", ToxicDirection.DOWNSTREAM, 2000);

        // When - Process order with network issues
        Order order = createTestOrder();
        String orderId = orderService.createOrder(order);

        // Then - Should handle gracefully with timeouts and retries
        await().atMost(Duration.ofSeconds(30)).untilAsserted(() -> {
            Order processedOrder = orderRepository.findById(orderId).orElseThrow();
            assertThat(processedOrder.getStatus()).isIn(OrderStatus.CONFIRMED, OrderStatus.CANCELLED);
        });
    }

    @Test
    void shouldRecoverFromServiceFailure() {
        // Test service recovery scenarios
    }
}
```

## Learning Goals

- **Microservices Architecture** - Service decomposition and communication
- **Distributed Transactions** - Saga pattern implementation
- **Event-Driven Architecture** - Asynchronous messaging with Kafka
- **Resilience Patterns** - Circuit breakers, retries, timeouts
- **Service Mesh** - Traffic management and observability
- **Contract Testing** - Consumer-driven contract testing
- **Chaos Engineering** - Failure injection and recovery testing
- **Distributed Tracing** - Request tracing across services
- **Performance Testing** - Load testing distributed systems

## TDD Steps for Distributed Systems

1. **Red**: Write contract tests for service interactions
2. **Green**: Implement service interfaces
3. **Red**: Write integration tests for message handling
4. **Green**: Implement message producers and consumers
5. **Red**: Write saga orchestration tests
6. **Green**: Implement saga pattern
7. **Red**: Write circuit breaker tests
8. **Green**: Add resilience patterns
9. **Red**: Write chaos engineering tests
10. **Green**: Add failure recovery mechanisms
11. **Refactor**: Optimize for performance and reliability

## Advanced Topics

- **Service Mesh** with Istio
- **Event Sourcing** with Axon Framework
- **CQRS** implementation
- **Distributed caching** with Hazelcast
- **API Versioning** strategies
- **Blue-Green Deployments**
- **Canary Releases**

---

## 问题描述

使用多个Spring Boot微服务构建分布式订单处理系统，包含异步通信、分布式事务、熔断器和全面的可观测性。

## 学习目标

- **微服务架构** - 服务分解和通信
- **分布式事务** - Saga模式实现
- **事件驱动架构** - Kafka异步消息传递
- **弹性模式** - 熔断器、重试、超时
- **服务网格** - 流量管理和可观测性
- **契约测试** - 消费者驱动的契约测试
- **混沌工程** - 故障注入和恢复测试
- **分布式追踪** - 跨服务请求追踪
- **性能测试** - 分布式系统负载测试