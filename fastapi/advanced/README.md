# FastAPI Advanced Problems

These advanced problems focus on distributed systems, microservices architecture, and enterprise-level patterns using FastAPI. Perfect for developers building scalable, production-ready systems.

[中文版本](#fastapi-高级问题)

## Problems

1. **[Microservice Order System](01-microservice-order-system.md)** - Distributed system with multiple FastAPI services
2. **[Event-Driven Architecture](02-event-driven-api.md)** - Event sourcing, CQRS, and async message processing
3. **[Multi-tenant SaaS API](03-multitenant-saas-api.md)** - Complex authorization, data isolation, and tenant management

## What You'll Learn

### Microservices Architecture
- **Service Decomposition** - Breaking monoliths into microservices
- **Inter-Service Communication** - HTTP APIs, message queues, event streams
- **Service Discovery** - Dynamic service registration and discovery
- **API Gateway** - Centralized routing, authentication, and rate limiting
- **Distributed Transactions** - Saga pattern and eventual consistency

### Event-Driven Systems
- **Event Sourcing** - Storing state as sequence of events
- **CQRS** - Command Query Responsibility Segregation
- **Message Brokers** - RabbitMQ, Apache Kafka integration
- **Event Streaming** - Real-time data processing and analytics
- **Eventual Consistency** - Handling distributed data consistency

### Enterprise Patterns
- **Multi-tenancy** - Shared infrastructure with data isolation
- **Clean Architecture** - Hexagonal architecture implementation
- **Domain-Driven Design** - Complex business domain modeling
- **SOLID Principles** - Advanced object-oriented design
- **Design Patterns** - Repository, Factory, Observer patterns

### Observability & Monitoring
- **Distributed Tracing** - OpenTelemetry and Jaeger integration
- **Metrics Collection** - Prometheus and Grafana dashboards
- **Centralized Logging** - ELK stack integration
- **Health Checks** - Service health monitoring and alerting
- **Performance Monitoring** - APM tools and optimization

### Security & Compliance
- **Zero Trust Architecture** - Service-to-service authentication
- **OAuth 2.0 / OpenID Connect** - Advanced authentication flows
- **API Security** - Rate limiting, CORS, input validation
- **Data Privacy** - GDPR compliance and data protection
- **Audit Logging** - Comprehensive audit trails

## Prerequisites

- Completion of [Intermediate Problems](../intermediate/)
- Understanding of distributed systems concepts
- Knowledge of message queues and event streaming
- Experience with Docker and containerization
- Familiarity with cloud platforms (AWS/GCP/Azure)
- Basic understanding of DevOps practices

## Problem Complexity

### Microservice Order System
**Complexity**: ⭐⭐⭐⭐⭐
**Focus Areas**:
- Multiple FastAPI services (Order, Payment, Inventory, Notification)
- Service-to-service communication patterns
- Distributed transaction management with Saga pattern
- API Gateway implementation
- Comprehensive testing across services

**Key Learning Points**:
- Microservices decomposition strategies
- Inter-service communication patterns
- Distributed system testing
- Service orchestration vs choreography

### Event-Driven Architecture
**Complexity**: ⭐⭐⭐⭐⭐
**Focus Areas**:
- Event sourcing implementation
- CQRS pattern with separate read/write models
- Message broker integration (RabbitMQ/Kafka)
- Event streaming and real-time processing
- Eventual consistency handling

**Key Learning Points**:
- Event-driven architecture principles
- Message broker integration
- Async processing patterns
- Data consistency in distributed systems

### Multi-tenant SaaS API
**Complexity**: ⭐⭐⭐⭐⭐
**Focus Areas**:
- Tenant isolation strategies
- Complex authorization and permissions
- Data partitioning and security
- Scalable architecture design
- Resource usage monitoring and billing

**Key Learning Points**:
- Multi-tenancy patterns
- Advanced security implementations
- Scalable system design
- Resource management and optimization

## Advanced Testing Patterns

### Microservice Testing
```python
import pytest
import asyncio
from httpx import AsyncClient
from testcontainers.compose import DockerCompose

@pytest.fixture(scope="session")
def docker_services():
    """Start all microservices for integration testing"""
    with DockerCompose(".", compose_file_name="docker-compose.test.yml") as compose:
        # Wait for services to be ready
        compose.wait_for("http://localhost:8001/health")  # Order service
        compose.wait_for("http://localhost:8002/health")  # Payment service
        compose.wait_for("http://localhost:8003/health")  # Inventory service
        yield compose

@pytest.mark.asyncio
async def test_order_workflow_integration(docker_services):
    """Test complete order workflow across multiple services"""
    async with AsyncClient() as client:
        # 1. Create order
        order_response = await client.post(
            "http://localhost:8001/orders",
            json={"product_id": 1, "quantity": 2, "customer_id": 123}
        )
        assert order_response.status_code == 201
        order_id = order_response.json()["id"]

        # 2. Verify inventory was reserved
        inventory_response = await client.get(
            f"http://localhost:8003/products/1/reservations"
        )
        assert inventory_response.status_code == 200
        reservations = inventory_response.json()
        assert any(r["order_id"] == order_id for r in reservations)

        # 3. Process payment
        payment_response = await client.post(
            "http://localhost:8002/payments",
            json={"order_id": order_id, "amount": 100.00, "payment_method": "card"}
        )
        assert payment_response.status_code == 201

        # 4. Verify order completion
        await asyncio.sleep(1)  # Wait for async processing
        final_order = await client.get(f"http://localhost:8001/orders/{order_id}")
        assert final_order.json()["status"] == "completed"
```

### Event-Driven Testing
```python
import pytest
from unittest.mock import AsyncMock, patch
from app.events import EventBus, OrderCreated, PaymentProcessed

@pytest.mark.asyncio
async def test_event_handling_chain():
    """Test event publishing and handling across services"""
    event_bus = EventBus()

    # Mock event handlers
    inventory_handler = AsyncMock()
    payment_handler = AsyncMock()
    notification_handler = AsyncMock()

    # Register handlers
    event_bus.subscribe(OrderCreated, inventory_handler)
    event_bus.subscribe(OrderCreated, payment_handler)
    event_bus.subscribe(PaymentProcessed, notification_handler)

    # Publish event
    order_event = OrderCreated(
        order_id="order_123",
        customer_id="customer_456",
        items=[{"product_id": 1, "quantity": 2}],
        total_amount=100.00
    )

    await event_bus.publish(order_event)

    # Verify handlers were called
    inventory_handler.assert_called_once_with(order_event)
    payment_handler.assert_called_once_with(order_event)

    # Simulate payment completion
    payment_event = PaymentProcessed(
        order_id="order_123",
        payment_id="payment_789",
        amount=100.00,
        status="success"
    )

    await event_bus.publish(payment_event)
    notification_handler.assert_called_once_with(payment_event)
```

### Performance Testing
```python
import asyncio
import aiohttp
import time
from concurrent.futures import ThreadPoolExecutor

async def load_test_api():
    """Load test API endpoints with concurrent requests"""
    async with aiohttp.ClientSession() as session:
        tasks = []

        # Create 100 concurrent requests
        for i in range(100):
            task = asyncio.create_task(
                session.post(
                    "http://localhost:8000/orders",
                    json={"product_id": 1, "quantity": 1, "customer_id": i}
                )
            )
            tasks.append(task)

        start_time = time.time()
        responses = await asyncio.gather(*tasks, return_exceptions=True)
        end_time = time.time()

        # Analyze results
        successful = sum(1 for r in responses if not isinstance(r, Exception) and r.status == 201)
        failed = len(responses) - successful
        duration = end_time - start_time

        print(f"Load Test Results:")
        print(f"Total requests: {len(responses)}")
        print(f"Successful: {successful}")
        print(f"Failed: {failed}")
        print(f"Duration: {duration:.2f} seconds")
        print(f"Requests per second: {len(responses) / duration:.2f}")

if __name__ == "__main__":
    asyncio.run(load_test_api())
```

## Production Patterns

### Circuit Breaker
```python
import asyncio
from enum import Enum
from typing import Callable, Any
from datetime import datetime, timedelta

class CircuitState(Enum):
    CLOSED = "closed"
    OPEN = "open"
    HALF_OPEN = "half_open"

class CircuitBreaker:
    def __init__(
        self,
        failure_threshold: int = 5,
        recovery_timeout: int = 60,
        expected_exception: Exception = Exception
    ):
        self.failure_threshold = failure_threshold
        self.recovery_timeout = recovery_timeout
        self.expected_exception = expected_exception

        self.failure_count = 0
        self.last_failure_time = None
        self.state = CircuitState.CLOSED

    async def call(self, func: Callable, *args, **kwargs) -> Any:
        if self.state == CircuitState.OPEN:
            if self._should_attempt_reset():
                self.state = CircuitState.HALF_OPEN
            else:
                raise Exception("Circuit breaker is OPEN")

        try:
            result = await func(*args, **kwargs)
            self._on_success()
            return result
        except self.expected_exception as e:
            self._on_failure()
            raise e

    def _should_attempt_reset(self) -> bool:
        return (
            self.last_failure_time and
            datetime.now() - self.last_failure_time >= timedelta(seconds=self.recovery_timeout)
        )

    def _on_success(self):
        self.failure_count = 0
        self.state = CircuitState.CLOSED

    def _on_failure(self):
        self.failure_count += 1
        self.last_failure_time = datetime.now()

        if self.failure_count >= self.failure_threshold:
            self.state = CircuitState.OPEN
```

### Retry with Exponential Backoff
```python
import asyncio
import random
from typing import Callable, Any

async def retry_with_exponential_backoff(
    func: Callable,
    max_retries: int = 3,
    base_delay: float = 1.0,
    max_delay: float = 60.0,
    exponential_base: float = 2.0,
    jitter: bool = True,
    *args,
    **kwargs
) -> Any:
    """
    Retry async function with exponential backoff
    """
    for attempt in range(max_retries + 1):
        try:
            return await func(*args, **kwargs)
        except Exception as e:
            if attempt == max_retries:
                raise e

            # Calculate delay with exponential backoff
            delay = min(base_delay * (exponential_base ** attempt), max_delay)

            # Add jitter to prevent thundering herd
            if jitter:
                delay = delay * (0.5 + random.random() * 0.5)

            await asyncio.sleep(delay)
```

### Distributed Tracing
```python
from opentelemetry import trace
from opentelemetry.exporter.jaeger.thrift import JaegerExporter
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import BatchSpanProcessor
from opentelemetry.instrumentation.fastapi import FastAPIInstrumentor
from opentelemetry.instrumentation.httpx import HTTPXClientInstrumentor

# Initialize tracing
trace.set_tracer_provider(TracerProvider())
tracer = trace.get_tracer(__name__)

# Configure Jaeger exporter
jaeger_exporter = JaegerExporter(
    agent_host_name="localhost",
    agent_port=6831,
)

span_processor = BatchSpanProcessor(jaeger_exporter)
trace.get_tracer_provider().add_span_processor(span_processor)

# Auto-instrument FastAPI and HTTPX
FastAPIInstrumentor.instrument_app(app)
HTTPXClientInstrumentor().instrument()

# Manual instrumentation example
@app.post("/orders")
async def create_order(order_data: OrderCreate):
    with tracer.start_as_current_span("create_order") as span:
        span.set_attribute("order.customer_id", order_data.customer_id)
        span.set_attribute("order.total_amount", order_data.total_amount)

        try:
            # Process order
            order = await order_service.create_order(order_data)
            span.set_attribute("order.id", order.id)
            span.set_status(trace.Status(trace.StatusCode.OK))
            return order
        except Exception as e:
            span.set_status(trace.Status(trace.StatusCode.ERROR, str(e)))
            raise
```

## Architecture Patterns

### Hexagonal Architecture
```python
# Domain layer - pure business logic
class Order:
    def __init__(self, customer_id: str, items: List[OrderItem]):
        self.id = None
        self.customer_id = customer_id
        self.items = items
        self.status = OrderStatus.PENDING
        self.total_amount = self._calculate_total()

    def _calculate_total(self) -> Decimal:
        return sum(item.price * item.quantity for item in self.items)

    def confirm(self):
        if self.status != OrderStatus.PENDING:
            raise InvalidOrderStateError("Cannot confirm non-pending order")
        self.status = OrderStatus.CONFIRMED

# Application layer - use cases
class CreateOrderUseCase:
    def __init__(
        self,
        order_repo: OrderRepository,
        inventory_service: InventoryService,
        event_publisher: EventPublisher
    ):
        self.order_repo = order_repo
        self.inventory_service = inventory_service
        self.event_publisher = event_publisher

    async def execute(self, command: CreateOrderCommand) -> Order:
        # Validate inventory
        await self.inventory_service.check_availability(command.items)

        # Create order
        order = Order(command.customer_id, command.items)
        order = await self.order_repo.save(order)

        # Publish event
        await self.event_publisher.publish(OrderCreated.from_order(order))

        return order

# Infrastructure layer - adapters
class SQLOrderRepository(OrderRepository):
    def __init__(self, db: Database):
        self.db = db

    async def save(self, order: Order) -> Order:
        # Implementation details
        pass

class HTTPInventoryService(InventoryService):
    def __init__(self, http_client: HTTPClient):
        self.http_client = http_client

    async def check_availability(self, items: List[OrderItem]) -> bool:
        # Implementation details
        pass
```

## Success Criteria

- [ ] Microservices architecture implemented
- [ ] Service-to-service communication working
- [ ] Event-driven patterns functional
- [ ] Distributed testing strategy
- [ ] Observability and monitoring setup
- [ ] Production-ready error handling
- [ ] Performance optimization implemented
- [ ] Security best practices followed

## Next Steps

After completing advanced problems:
1. Study cloud-native patterns (Kubernetes, service mesh)
2. Learn about serverless architectures
3. Explore reactive programming patterns
4. Master distributed system debugging
5. Implement chaos engineering practices

---

# FastAPI 高级问题

这些高级问题专注于分布式系统、微服务架构和使用FastAPI的企业级模式。非常适合构建可扩展、生产就绪系统的开发者。

## 问题列表

1. **[微服务订单系统](01-microservice-order-system.md)** - 多个FastAPI服务的分布式系统
2. **[事件驱动架构](02-event-driven-api.md)** - 事件溯源、CQRS和异步消息处理
3. **[多租户SaaS API](03-multitenant-saas-api.md)** - 复杂授权、数据隔离和租户管理

## 你将学到什么

### 微服务架构
- **服务分解** - 将单体应用分解为微服务
- **服务间通信** - HTTP API、消息队列、事件流
- **服务发现** - 动态服务注册和发现
- **API网关** - 集中式路由、认证和限流
- **分布式事务** - Saga模式和最终一致性

### 事件驱动系统
- **事件溯源** - 将状态存储为事件序列
- **CQRS** - 命令查询职责分离
- **消息代理** - RabbitMQ、Apache Kafka集成
- **事件流** - 实时数据处理和分析
- **最终一致性** - 处理分布式数据一致性

## 前置条件

- 完成[中级问题](../intermediate/)
- 理解分布式系统概念
- 消息队列和事件流知识
- Docker和容器化经验
- 熟悉云平台（AWS/GCP/Azure）
- DevOps实践基本理解