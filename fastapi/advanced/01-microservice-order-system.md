# Microservice Order System

Build a distributed e-commerce order system using multiple FastAPI microservices with inter-service communication, distributed transactions, and comprehensive testing. This problem demonstrates microservices architecture patterns and distributed system challenges.

[中文版本](#微服务订单系统)

## Problem Description

Create a distributed order processing system composed of multiple FastAPI microservices:
- **Order Service** - Manages order lifecycle and orchestration
- **Payment Service** - Handles payment processing and validation
- **Inventory Service** - Manages product inventory and reservations
- **Notification Service** - Sends notifications to customers
- **API Gateway** - Routes requests and handles cross-cutting concerns

The system should implement proper microservices patterns including service discovery, circuit breakers, distributed tracing, and comprehensive testing.

## Requirements

### Functional Requirements
1. **Order Management** - Create, update, cancel orders
2. **Payment Processing** - Process payments with multiple providers
3. **Inventory Management** - Track stock levels and reservations
4. **Notification System** - Email/SMS notifications for order events
5. **Order Tracking** - Real-time order status updates
6. **Saga Pattern** - Distributed transaction management
7. **API Gateway** - Centralized routing and authentication

### Technical Requirements
- Multiple FastAPI services with independent databases
- Service-to-service communication (HTTP/gRPC)
- Message queue integration (RabbitMQ/Redis)
- Distributed tracing with OpenTelemetry
- Circuit breaker pattern implementation
- Comprehensive testing strategy
- Docker containerization

## System Architecture

```
┌─────────────┐    ┌─────────────────┐    ┌─────────────────┐
│  API Gateway│────│  Order Service  │────│ Payment Service │
└─────────────┘    └─────────────────┘    └─────────────────┘
       │                     │                      │
       │            ┌─────────────────┐             │
       └────────────│Inventory Service│─────────────┘
                    └─────────────────┘
                             │
                    ┌─────────────────┐
                    │Notification Svc │
                    └─────────────────┘
                             │
                    ┌─────────────────┐
                    │  Message Queue  │
                    └─────────────────┘
```

## Service Specifications

### Order Service (Port 8001)

**Responsibilities:**
- Order lifecycle management
- Saga orchestration
- Order status tracking
- Customer order history

**Endpoints:**
```
POST   /orders                 # Create new order
GET    /orders/{order_id}      # Get order details
PUT    /orders/{order_id}      # Update order
DELETE /orders/{order_id}      # Cancel order
GET    /orders/customer/{id}   # Get customer orders
GET    /orders/{id}/status     # Get order status
```

**Database Schema:**
```sql
CREATE TABLE orders (
    id UUID PRIMARY KEY,
    customer_id UUID NOT NULL,
    status VARCHAR(20) NOT NULL,
    total_amount DECIMAL(10,2) NOT NULL,
    currency VARCHAR(3) DEFAULT 'USD',
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE order_items (
    id UUID PRIMARY KEY,
    order_id UUID REFERENCES orders(id),
    product_id UUID NOT NULL,
    quantity INTEGER NOT NULL,
    unit_price DECIMAL(10,2) NOT NULL,
    total_price DECIMAL(10,2) NOT NULL
);

CREATE TABLE order_events (
    id UUID PRIMARY KEY,
    order_id UUID REFERENCES orders(id),
    event_type VARCHAR(50) NOT NULL,
    event_data JSONB,
    created_at TIMESTAMP DEFAULT NOW()
);
```

### Payment Service (Port 8002)

**Responsibilities:**
- Payment processing
- Payment method validation
- Refund handling
- Payment status tracking

**Endpoints:**
```
POST   /payments              # Process payment
GET    /payments/{payment_id} # Get payment details
POST   /payments/{id}/refund  # Process refund
GET    /payments/order/{id}   # Get order payments
```

### Inventory Service (Port 8003)

**Responsibilities:**
- Stock level management
- Inventory reservations
- Product availability checks
- Warehouse management

**Endpoints:**
```
GET    /products/{id}/stock   # Check stock level
POST   /reservations          # Reserve inventory
PUT    /reservations/{id}     # Update reservation
DELETE /reservations/{id}     # Cancel reservation
GET    /products              # List products
```

### Notification Service (Port 8004)

**Responsibilities:**
- Email notifications
- SMS notifications
- Push notifications
- Notification templates

**Endpoints:**
```
POST   /notifications/email   # Send email
POST   /notifications/sms     # Send SMS
GET    /notifications/{id}    # Get notification status
GET    /templates             # List templates
```

## Test Examples

### Service Integration Tests
```python
import pytest
import asyncio
from httpx import AsyncClient
from testcontainers.compose import DockerCompose
from testcontainers.postgres import PostgresContainer

class TestOrderWorkflow:
    @pytest.fixture(scope="class")
    def docker_services(self):
        """Start all microservices using docker-compose"""
        with DockerCompose(".", compose_file_name="docker-compose.test.yml") as compose:
            # Wait for all services to be healthy
            compose.wait_for("http://localhost:8001/health")  # Order service
            compose.wait_for("http://localhost:8002/health")  # Payment service
            compose.wait_for("http://localhost:8003/health")  # Inventory service
            compose.wait_for("http://localhost:8004/health")  # Notification service
            yield compose

    @pytest.mark.asyncio
    async def test_successful_order_flow(self, docker_services):
        """Test complete order workflow from creation to completion"""
        async with AsyncClient() as client:
            # 1. Check product availability
            product_id = "product-123"
            inventory_response = await client.get(
                f"http://localhost:8003/products/{product_id}/stock"
            )
            assert inventory_response.status_code == 200
            assert inventory_response.json()["available"] >= 2

            # 2. Create order
            order_data = {
                "customer_id": "customer-456",
                "items": [
                    {
                        "product_id": product_id,
                        "quantity": 2,
                        "unit_price": 29.99
                    }
                ]
            }

            order_response = await client.post(
                "http://localhost:8001/orders",
                json=order_data
            )
            assert order_response.status_code == 201
            order = order_response.json()
            order_id = order["id"]

            # 3. Verify inventory reservation
            await asyncio.sleep(1)  # Wait for async processing
            reservation_response = await client.get(
                f"http://localhost:8003/reservations?order_id={order_id}"
            )
            assert reservation_response.status_code == 200
            assert len(reservation_response.json()) > 0

            # 4. Process payment
            payment_data = {
                "order_id": order_id,
                "amount": order["total_amount"],
                "payment_method": {
                    "type": "credit_card",
                    "card_number": "4111111111111111",
                    "expiry": "12/25",
                    "cvv": "123"
                }
            }

            payment_response = await client.post(
                "http://localhost:8002/payments",
                json=payment_data
            )
            assert payment_response.status_code == 201
            payment = payment_response.json()
            assert payment["status"] == "success"

            # 5. Verify order completion
            await asyncio.sleep(2)  # Wait for saga completion
            final_order_response = await client.get(
                f"http://localhost:8001/orders/{order_id}"
            )
            final_order = final_order_response.json()
            assert final_order["status"] == "completed"

            # 6. Verify notification was sent
            notifications_response = await client.get(
                f"http://localhost:8004/notifications?order_id={order_id}"
            )
            assert notifications_response.status_code == 200
            notifications = notifications_response.json()
            assert any(n["type"] == "order_confirmation" for n in notifications)

    @pytest.mark.asyncio
    async def test_payment_failure_rollback(self, docker_services):
        """Test order rollback when payment fails"""
        async with AsyncClient() as client:
            # Create order
            order_data = {
                "customer_id": "customer-789",
                "items": [{"product_id": "product-456", "quantity": 1, "unit_price": 19.99}]
            }

            order_response = await client.post(
                "http://localhost:8001/orders",
                json=order_data
            )
            order_id = order_response.json()["id"]

            # Attempt payment with invalid card
            payment_data = {
                "order_id": order_id,
                "amount": 19.99,
                "payment_method": {
                    "type": "credit_card",
                    "card_number": "4000000000000002",  # Declined card
                    "expiry": "12/25",
                    "cvv": "123"
                }
            }

            payment_response = await client.post(
                "http://localhost:8002/payments",
                json=payment_data
            )
            assert payment_response.status_code == 400
            assert payment_response.json()["status"] == "failed"

            # Verify order was cancelled
            await asyncio.sleep(2)  # Wait for saga rollback
            order_response = await client.get(f"http://localhost:8001/orders/{order_id}")
            assert order_response.json()["status"] == "cancelled"

            # Verify inventory reservation was released
            reservation_response = await client.get(
                f"http://localhost:8003/reservations?order_id={order_id}"
            )
            reservations = reservation_response.json()
            assert all(r["status"] == "cancelled" for r in reservations)

    @pytest.mark.asyncio
    async def test_insufficient_inventory(self, docker_services):
        """Test order failure when inventory is insufficient"""
        async with AsyncClient() as client:
            order_data = {
                "customer_id": "customer-999",
                "items": [
                    {
                        "product_id": "product-789",
                        "quantity": 1000,  # Excessive quantity
                        "unit_price": 9.99
                    }
                ]
            }

            order_response = await client.post(
                "http://localhost:8001/orders",
                json=order_data
            )

            # Order creation should fail due to insufficient inventory
            assert order_response.status_code == 400
            assert "insufficient inventory" in order_response.json()["detail"].lower()
```

### Service Unit Tests
```python
import pytest
from unittest.mock import AsyncMock, patch
from app.services.order_service import OrderService
from app.models.order import Order, OrderStatus
from app.saga.order_saga import OrderSaga

class TestOrderService:
    @pytest.fixture
    def order_service(self):
        return OrderService(
            order_repository=AsyncMock(),
            inventory_client=AsyncMock(),
            payment_client=AsyncMock(),
            notification_client=AsyncMock()
        )

    @pytest.mark.asyncio
    async def test_create_order_success(self, order_service):
        # Mock dependencies
        order_service.inventory_client.check_availability.return_value = True
        order_service.inventory_client.reserve_items.return_value = {"reservation_id": "res-123"}
        order_service.order_repository.save.return_value = Order(
            id="order-123",
            customer_id="customer-456",
            status=OrderStatus.PENDING
        )

        # Test order creation
        order_data = {
            "customer_id": "customer-456",
            "items": [{"product_id": "product-123", "quantity": 2, "unit_price": 29.99}]
        }

        result = await order_service.create_order(order_data)

        assert result.id == "order-123"
        assert result.status == OrderStatus.PENDING
        order_service.inventory_client.check_availability.assert_called_once()
        order_service.inventory_client.reserve_items.assert_called_once()

    @pytest.mark.asyncio
    async def test_create_order_insufficient_inventory(self, order_service):
        # Mock insufficient inventory
        order_service.inventory_client.check_availability.return_value = False

        order_data = {
            "customer_id": "customer-456",
            "items": [{"product_id": "product-123", "quantity": 1000, "unit_price": 29.99}]
        }

        with pytest.raises(InsufficientInventoryError):
            await order_service.create_order(order_data)

        order_service.inventory_client.reserve_items.assert_not_called()

class TestOrderSaga:
    @pytest.fixture
    def saga(self):
        return OrderSaga(
            order_service=AsyncMock(),
            payment_service=AsyncMock(),
            inventory_service=AsyncMock(),
            notification_service=AsyncMock()
        )

    @pytest.mark.asyncio
    async def test_saga_success_flow(self, saga):
        # Mock successful responses
        saga.inventory_service.reserve_items.return_value = {"status": "success"}
        saga.payment_service.process_payment.return_value = {"status": "success"}
        saga.notification_service.send_confirmation.return_value = {"status": "sent"}

        order = Order(id="order-123", customer_id="customer-456")

        result = await saga.execute(order)

        assert result["status"] == "completed"
        saga.inventory_service.reserve_items.assert_called_once()
        saga.payment_service.process_payment.assert_called_once()
        saga.notification_service.send_confirmation.assert_called_once()

    @pytest.mark.asyncio
    async def test_saga_rollback_on_payment_failure(self, saga):
        # Mock payment failure
        saga.inventory_service.reserve_items.return_value = {"status": "success"}
        saga.payment_service.process_payment.side_effect = PaymentFailedError("Card declined")
        saga.inventory_service.release_reservation.return_value = {"status": "released"}

        order = Order(id="order-123", customer_id="customer-456")

        with pytest.raises(PaymentFailedError):
            await saga.execute(order)

        # Verify rollback was called
        saga.inventory_service.release_reservation.assert_called_once()
        saga.notification_service.send_confirmation.assert_not_called()
```

## Implementation Guide

### Step 1: Order Service Implementation
```python
from fastapi import FastAPI, HTTPException, Depends
from sqlalchemy.orm import Session
from app.models import Order, OrderItem, OrderStatus
from app.schemas import OrderCreate, OrderResponse
from app.services import OrderService
from app.database import get_db
from app.saga import OrderSaga

app = FastAPI(title="Order Service", version="1.0.0")

@app.post("/orders", response_model=OrderResponse, status_code=201)
async def create_order(
    order_data: OrderCreate,
    db: Session = Depends(get_db),
    order_service: OrderService = Depends()
):
    try:
        order = await order_service.create_order(order_data)

        # Start saga for order processing
        saga = OrderSaga()
        await saga.execute(order)

        return OrderResponse.from_orm(order)
    except InsufficientInventoryError as e:
        raise HTTPException(status_code=400, detail=str(e))
    except Exception as e:
        raise HTTPException(status_code=500, detail="Internal server error")

@app.get("/orders/{order_id}", response_model=OrderResponse)
async def get_order(
    order_id: str,
    db: Session = Depends(get_db),
    order_service: OrderService = Depends()
):
    order = await order_service.get_order(order_id)
    if not order:
        raise HTTPException(status_code=404, detail="Order not found")
    return OrderResponse.from_orm(order)

@app.put("/orders/{order_id}/cancel")
async def cancel_order(
    order_id: str,
    order_service: OrderService = Depends()
):
    try:
        await order_service.cancel_order(order_id)
        return {"message": "Order cancelled successfully"}
    except OrderNotFoundError:
        raise HTTPException(status_code=404, detail="Order not found")
    except InvalidOrderStateError as e:
        raise HTTPException(status_code=400, detail=str(e))

@app.get("/health")
async def health_check():
    return {"status": "healthy", "service": "order-service"}
```

### Step 2: Saga Pattern Implementation
```python
import asyncio
from enum import Enum
from typing import Dict, Any, List
from dataclasses import dataclass
from app.exceptions import SagaRollbackError

class SagaStepStatus(Enum):
    PENDING = "pending"
    COMPLETED = "completed"
    FAILED = "failed"
    COMPENSATED = "compensated"

@dataclass
class SagaStep:
    name: str
    execute_func: callable
    compensate_func: callable
    status: SagaStepStatus = SagaStepStatus.PENDING
    result: Any = None
    error: Exception = None

class OrderSaga:
    def __init__(self):
        self.steps: List[SagaStep] = []
        self.completed_steps: List[SagaStep] = []

    def add_step(self, name: str, execute_func: callable, compensate_func: callable):
        step = SagaStep(name, execute_func, compensate_func)
        self.steps.append(step)

    async def execute(self, order: Order) -> Dict[str, Any]:
        """Execute saga steps in sequence"""
        try:
            for step in self.steps:
                await self._execute_step(step, order)
                self.completed_steps.append(step)

            return {"status": "completed", "order_id": order.id}

        except Exception as e:
            await self._compensate()
            raise SagaRollbackError(f"Saga failed: {str(e)}")

    async def _execute_step(self, step: SagaStep, order: Order):
        """Execute a single saga step"""
        try:
            step.result = await step.execute_func(order)
            step.status = SagaStepStatus.COMPLETED
        except Exception as e:
            step.error = e
            step.status = SagaStepStatus.FAILED
            raise e

    async def _compensate(self):
        """Compensate completed steps in reverse order"""
        for step in reversed(self.completed_steps):
            try:
                await step.compensate_func(step.result)
                step.status = SagaStepStatus.COMPENSATED
            except Exception as e:
                # Log compensation error but continue
                print(f"Compensation failed for step {step.name}: {str(e)}")

# Saga factory for order processing
def create_order_saga(
    inventory_service,
    payment_service,
    notification_service
) -> OrderSaga:
    saga = OrderSaga()

    # Step 1: Reserve inventory
    saga.add_step(
        "reserve_inventory",
        execute_func=lambda order: inventory_service.reserve_items(order.items),
        compensate_func=lambda result: inventory_service.release_reservation(result["reservation_id"])
    )

    # Step 2: Process payment
    saga.add_step(
        "process_payment",
        execute_func=lambda order: payment_service.process_payment(order),
        compensate_func=lambda result: payment_service.refund_payment(result["payment_id"])
    )

    # Step 3: Send confirmation
    saga.add_step(
        "send_notification",
        execute_func=lambda order: notification_service.send_order_confirmation(order),
        compensate_func=lambda result: None  # No compensation needed for notifications
    )

    return saga
```

### Step 3: Service Communication
```python
import httpx
from typing import Dict, Any
from app.config import settings

class InventoryServiceClient:
    def __init__(self):
        self.base_url = settings.INVENTORY_SERVICE_URL
        self.timeout = httpx.Timeout(30.0)

    async def check_availability(self, items: List[Dict]) -> bool:
        """Check if items are available in inventory"""
        async with httpx.AsyncClient(timeout=self.timeout) as client:
            response = await client.post(
                f"{self.base_url}/check-availability",
                json={"items": items}
            )

            if response.status_code == 200:
                return response.json()["available"]
            elif response.status_code == 400:
                return False
            else:
                raise InventoryServiceError(f"Inventory service error: {response.status_code}")

    async def reserve_items(self, items: List[Dict]) -> Dict[str, Any]:
        """Reserve items in inventory"""
        async with httpx.AsyncClient(timeout=self.timeout) as client:
            response = await client.post(
                f"{self.base_url}/reservations",
                json={"items": items}
            )

            if response.status_code == 201:
                return response.json()
            else:
                raise InventoryServiceError(f"Failed to reserve items: {response.status_code}")

    async def release_reservation(self, reservation_id: str) -> Dict[str, Any]:
        """Release inventory reservation"""
        async with httpx.AsyncClient(timeout=self.timeout) as client:
            response = await client.delete(
                f"{self.base_url}/reservations/{reservation_id}"
            )

            if response.status_code == 200:
                return response.json()
            else:
                raise InventoryServiceError(f"Failed to release reservation: {response.status_code}")

class PaymentServiceClient:
    def __init__(self):
        self.base_url = settings.PAYMENT_SERVICE_URL
        self.timeout = httpx.Timeout(60.0)  # Longer timeout for payments

    async def process_payment(self, order: Order) -> Dict[str, Any]:
        """Process payment for order"""
        payment_data = {
            "order_id": order.id,
            "amount": order.total_amount,
            "currency": order.currency,
            "customer_id": order.customer_id
        }

        async with httpx.AsyncClient(timeout=self.timeout) as client:
            response = await client.post(
                f"{self.base_url}/payments",
                json=payment_data
            )

            if response.status_code == 201:
                return response.json()
            elif response.status_code == 400:
                error_data = response.json()
                raise PaymentFailedError(error_data.get("detail", "Payment failed"))
            else:
                raise PaymentServiceError(f"Payment service error: {response.status_code}")

    async def refund_payment(self, payment_id: str) -> Dict[str, Any]:
        """Refund a payment"""
        async with httpx.AsyncClient(timeout=self.timeout) as client:
            response = await client.post(
                f"{self.base_url}/payments/{payment_id}/refund"
            )

            if response.status_code == 200:
                return response.json()
            else:
                raise PaymentServiceError(f"Failed to refund payment: {response.status_code}")
```

### Step 4: Circuit Breaker Implementation
```python
import asyncio
from datetime import datetime, timedelta
from enum import Enum
from typing import Callable, Any

class CircuitState(Enum):
    CLOSED = "closed"
    OPEN = "open"
    HALF_OPEN = "half_open"

class CircuitBreaker:
    def __init__(
        self,
        failure_threshold: int = 5,
        recovery_timeout: int = 60,
        expected_exception: type = Exception
    ):
        self.failure_threshold = failure_threshold
        self.recovery_timeout = recovery_timeout
        self.expected_exception = expected_exception

        self.failure_count = 0
        self.last_failure_time = None
        self.state = CircuitState.CLOSED

    async def call(self, func: Callable, *args, **kwargs) -> Any:
        """Execute function with circuit breaker protection"""
        if self.state == CircuitState.OPEN:
            if self._should_attempt_reset():
                self.state = CircuitState.HALF_OPEN
            else:
                raise CircuitBreakerOpenError("Circuit breaker is OPEN")

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

# Usage with service clients
class ResilientInventoryClient(InventoryServiceClient):
    def __init__(self):
        super().__init__()
        self.circuit_breaker = CircuitBreaker(
            failure_threshold=3,
            recovery_timeout=30,
            expected_exception=InventoryServiceError
        )

    async def check_availability(self, items: List[Dict]) -> bool:
        return await self.circuit_breaker.call(
            super().check_availability,
            items
        )
```

## Success Criteria

- [ ] Multiple FastAPI services communicating properly
- [ ] Saga pattern for distributed transactions
- [ ] Circuit breaker implementation working
- [ ] Comprehensive integration testing
- [ ] Service discovery and health checks
- [ ] Distributed tracing implemented
- [ ] Error handling and rollback mechanisms
- [ ] Performance monitoring and metrics

## Extensions

1. **Service Mesh** - Istio integration for advanced traffic management
2. **Event Sourcing** - Complete event-driven architecture
3. **CQRS Implementation** - Separate read/write models
4. **Kubernetes Deployment** - Cloud-native deployment
5. **API Versioning** - Backward-compatible API evolution

---

# 微服务订单系统

使用多个FastAPI微服务构建分布式电商订单系统，具有服务间通信、分布式事务和综合测试。这个问题演示了微服务架构模式和分布式系统挑战。

## 问题描述

创建一个由多个FastAPI微服务组成的分布式订单处理系统：
- **订单服务** - 管理订单生命周期和编排
- **支付服务** - 处理支付处理和验证
- **库存服务** - 管理产品库存和预留
- **通知服务** - 向客户发送通知
- **API网关** - 路由请求和处理横切关注点

系统应实现适当的微服务模式，包括服务发现、熔断器、分布式追踪和综合测试。

## 成功标准

- [ ] 多个FastAPI服务正确通信
- [ ] 分布式事务的Saga模式
- [ ] 熔断器实现工作
- [ ] 全面的集成测试
- [ ] 服务发现和健康检查
- [ ] 实施分布式追踪
- [ ] 错误处理和回滚机制
- [ ] 性能监控和指标