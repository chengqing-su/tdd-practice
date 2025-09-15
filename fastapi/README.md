# FastAPI TDD Practice Problems

This section contains Test-Driven Development (TDD) practice problems specifically designed for FastAPI applications. Learn to build robust, testable APIs using modern Python web development practices.

[中文版本](#fastapi-tdd-练习题)

## 🚀 Overview

FastAPI is a modern, high-performance web framework for building APIs with Python. These problems will help you master TDD practices while learning FastAPI's powerful features including automatic API documentation, request/response validation, dependency injection, and async support.

## 📚 Problem Categories

### [Beginner Problems](beginner/)
Perfect for developers new to FastAPI or API testing:

1. **[Simple REST API](beginner/01-simple-rest-api.md)** - Basic CRUD operations with in-memory storage
2. **[User Authentication API](beginner/02-user-auth-api.md)** - JWT authentication and user management
3. **[Todo API with Validation](beginner/03-todo-api-validation.md)** - Pydantic models and request validation
4. **[File Upload API](beginner/04-file-upload-api.md)** - File handling and multipart requests

### [Intermediate Problems](intermediate/)
Advanced FastAPI features and database integration:

1. **[Blog API with Database](intermediate/01-blog-api-database.md)** - SQLAlchemy ORM and database testing
2. **[E-commerce Product API](intermediate/02-ecommerce-product-api.md)** - Complex business logic and relationships
3. **[Task Queue API](intermediate/03-task-queue-api.md)** - Background tasks and async processing
4. **[Real-time Chat API](intermediate/04-realtime-chat-api.md)** - WebSockets and real-time communication

### [Advanced Problems](advanced/)
Enterprise-level patterns and distributed systems:

1. **[Microservice Order System](advanced/01-microservice-order-system.md)** - Multiple FastAPI services with communication
2. **[Event-Driven Architecture](advanced/02-event-driven-api.md)** - Event sourcing and CQRS patterns
3. **[Multi-tenant SaaS API](advanced/03-multitenant-saas-api.md)** - Complex authorization and data isolation

## 🛠️ Key Technologies & Concepts

### FastAPI Features
- **Automatic API Documentation** - Swagger/OpenAPI integration
- **Pydantic Models** - Request/response validation and serialization
- **Dependency Injection** - Clean architecture and testability
- **Async Support** - High-performance asynchronous operations
- **WebSockets** - Real-time bidirectional communication
- **Background Tasks** - Asynchronous job processing

### Testing Tools
- **pytest** - Python testing framework
- **httpx** - Async HTTP client for testing
- **TestClient** - FastAPI's built-in testing client
- **pytest-asyncio** - Testing async code
- **factory_boy** - Test data generation
- **freezegun** - Time mocking for tests

### Database & ORM
- **SQLAlchemy** - Python SQL toolkit and ORM
- **Alembic** - Database migration tool
- **pytest-postgresql** - Testing with real PostgreSQL
- **SQLModel** - FastAPI-native ORM (Pydantic + SQLAlchemy)

### Authentication & Security
- **JWT** - JSON Web Tokens for stateless auth
- **OAuth2** - Standard authorization framework
- **Password Hashing** - Secure password storage
- **API Key Management** - Service-to-service authentication
- **Rate Limiting** - API abuse prevention

## 🧪 FastAPI Testing Patterns

### Basic API Testing
```python
from fastapi.testclient import TestClient
from myapp import app

client = TestClient(app)

def test_create_user():
    response = client.post(
        "/users/",
        json={"username": "testuser", "email": "test@example.com"}
    )
    assert response.status_code == 201
    assert response.json()["username"] == "testuser"
```

### Async Testing
```python
import pytest
from httpx import AsyncClient
from myapp import app

@pytest.mark.asyncio
async def test_async_endpoint():
    async with AsyncClient(app=app, base_url="http://test") as ac:
        response = await ac.get("/async-endpoint")
    assert response.status_code == 200
```

### Database Testing
```python
import pytest
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker
from myapp.database import Base, get_db
from myapp import app

@pytest.fixture
def db_session():
    engine = create_engine("sqlite:///:memory:")
    Base.metadata.create_all(engine)
    SessionLocal = sessionmaker(bind=engine)
    session = SessionLocal()

    def override_get_db():
        try:
            yield session
        finally:
            session.close()

    app.dependency_overrides[get_db] = override_get_db
    yield session
    session.close()
```

### Authentication Testing
```python
def test_protected_endpoint_requires_auth():
    response = client.get("/protected")
    assert response.status_code == 401

def test_protected_endpoint_with_token():
    token = create_test_token(user_id=1)
    headers = {"Authorization": f"Bearer {token}"}
    response = client.get("/protected", headers=headers)
    assert response.status_code == 200
```

## 📋 Prerequisites

### Python Knowledge
- Python 3.8+ syntax and features
- Object-oriented programming
- Async/await patterns
- Type hints and annotations

### Web Development Concepts
- HTTP methods and status codes
- REST API design principles
- JSON data format
- Authentication and authorization
- Database relationships

### Testing Experience
- Basic pytest knowledge
- Unit vs integration testing
- Mocking and fixtures
- Test data management

## 🎯 Learning Objectives

### API Development Skills
- Design RESTful APIs following best practices
- Implement proper HTTP status codes and error handling
- Use Pydantic for data validation and serialization
- Handle file uploads and multipart requests
- Implement authentication and authorization

### Testing Mastery
- Write comprehensive API tests using TestClient
- Test async endpoints and background tasks
- Mock external dependencies and services
- Test database operations and transactions
- Verify WebSocket communication

### Architecture Patterns
- Apply dependency injection for testable code
- Implement repository pattern for data access
- Use service layer for business logic
- Handle cross-cutting concerns (logging, monitoring)
- Design for scalability and maintainability

## 🔧 Development Setup

### Required Dependencies
```bash
# Core FastAPI dependencies
pip install fastapi uvicorn python-multipart

# Testing dependencies
pip install pytest httpx pytest-asyncio

# Database dependencies (for intermediate+)
pip install sqlalchemy alembic psycopg2-binary

# Authentication dependencies
pip install python-jose[cryptography] passlib[bcrypt]

# Additional utilities
pip install factory-boy freezegun pytest-postgresql
```

### Project Structure
```
fastapi-project/
├── app/
│   ├── __init__.py
│   ├── main.py              # FastAPI app instance
│   ├── models/              # Pydantic models
│   ├── api/                 # API routes
│   ├── core/                # Configuration, security
│   ├── db/                  # Database models, session
│   └── services/            # Business logic
├── tests/
│   ├── __init__.py
│   ├── conftest.py          # Pytest fixtures
│   ├── test_api/            # API endpoint tests
│   └── test_services/       # Service layer tests
├── requirements.txt
└── README.md
```

## 🚦 TDD Workflow for FastAPI

### 1. Red Phase - Write Failing Test
```python
def test_create_user_returns_created_user():
    user_data = {"username": "testuser", "email": "test@example.com"}
    response = client.post("/users/", json=user_data)

    assert response.status_code == 201
    assert response.json()["username"] == "testuser"
    assert "id" in response.json()
```

### 2. Green Phase - Minimal Implementation
```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

class UserCreate(BaseModel):
    username: str
    email: str

class User(UserCreate):
    id: int

@app.post("/users/", response_model=User, status_code=201)
def create_user(user: UserCreate):
    return User(id=1, **user.dict())
```

### 3. Refactor Phase - Improve Code
```python
# Add proper data storage, validation, error handling
from fastapi import FastAPI, HTTPException
from sqlalchemy.orm import Session
from .database import get_db
from .models import User
from .schemas import UserCreate, UserResponse

@app.post("/users/", response_model=UserResponse, status_code=201)
def create_user(user: UserCreate, db: Session = Depends(get_db)):
    if get_user_by_email(db, user.email):
        raise HTTPException(400, "Email already registered")

    return create_user_in_db(db, user)
```

## 🎓 Best Practices

### API Design
- Use appropriate HTTP status codes
- Implement consistent error responses
- Version your APIs properly
- Document with clear descriptions
- Follow RESTful conventions

### Testing Strategy
- Test both success and error scenarios
- Use factory patterns for test data
- Test edge cases and validation
- Separate unit and integration tests
- Mock external dependencies

### Code Organization
- Separate concerns (routes, models, services)
- Use dependency injection
- Implement proper error handling
- Add comprehensive logging
- Follow type hints consistently

### Performance & Security
- Implement proper authentication
- Use async patterns appropriately
- Add rate limiting for protection
- Validate all inputs thoroughly
- Handle sensitive data securely

## 🔄 Progressive Learning Path

### Phase 1: FastAPI Fundamentals (Beginner)
1. Complete **Simple REST API** - Learn basic FastAPI syntax
2. Work through **User Authentication API** - Master security patterns
3. Build **Todo API with Validation** - Understand Pydantic models
4. Implement **File Upload API** - Handle multipart requests

### Phase 2: Database Integration (Intermediate)
1. **Blog API with Database** - Learn SQLAlchemy with FastAPI
2. **E-commerce Product API** - Handle complex relationships
3. **Task Queue API** - Master background processing
4. **Real-time Chat API** - Implement WebSocket communication

### Phase 3: Enterprise Patterns (Advanced)
1. **Microservice Order System** - Build distributed systems
2. **Event-Driven Architecture** - Implement CQRS and event sourcing
3. **Multi-tenant SaaS API** - Handle complex authorization

## 🤝 Contributing

To contribute FastAPI problems:
1. Follow the established problem format
2. Include comprehensive test examples
3. Provide both English and Chinese versions
4. Test all code examples thoroughly
5. Focus on real-world scenarios

---

# FastAPI TDD 练习题

本节包含专为FastAPI应用程序设计的测试驱动开发(TDD)练习题。学习使用现代Python Web开发实践构建健壮、可测试的API。

## 🚀 概览

FastAPI是用Python构建API的现代高性能Web框架。这些问题将帮助您在学习FastAPI强大功能的同时掌握TDD实践，包括自动API文档、请求/响应验证、依赖注入和异步支持。

## 📚 问题分类

### [初级问题](beginner/)
适合FastAPI或API测试新手的开发者：

1. **[简单REST API](beginner/01-simple-rest-api.md)** - 内存存储的基本CRUD操作
2. **[用户认证API](beginner/02-user-auth-api.md)** - JWT认证和用户管理
3. **[带验证的Todo API](beginner/03-todo-api-validation.md)** - Pydantic模型和请求验证
4. **[文件上传API](beginner/04-file-upload-api.md)** - 文件处理和多部分请求

### [中级问题](intermediate/)
高级FastAPI功能和数据库集成：

1. **[带数据库的博客API](intermediate/01-blog-api-database.md)** - SQLAlchemy ORM和数据库测试
2. **[电商产品API](intermediate/02-ecommerce-product-api.md)** - 复杂业务逻辑和关系
3. **[任务队列API](intermediate/03-task-queue-api.md)** - 后台任务和异步处理
4. **[实时聊天API](intermediate/04-realtime-chat-api.md)** - WebSockets和实时通信

### [高级问题](advanced/)
企业级模式和分布式系统：

1. **[微服务订单系统](advanced/01-microservice-order-system.md)** - 多个FastAPI服务通信
2. **[事件驱动架构](advanced/02-event-driven-api.md)** - 事件溯源和CQRS模式
3. **[多租户SaaS API](advanced/03-multitenant-saas-api.md)** - 复杂授权和数据隔离

## 🧪 FastAPI测试模式

### 基本API测试
```python
from fastapi.testclient import TestClient
from myapp import app

client = TestClient(app)

def test_create_user():
    response = client.post(
        "/users/",
        json={"username": "testuser", "email": "test@example.com"}
    )
    assert response.status_code == 201
    assert response.json()["username"] == "testuser"
```

## 🎯 学习目标

### API开发技能
- 遵循最佳实践设计RESTful API
- 实现适当的HTTP状态码和错误处理
- 使用Pydantic进行数据验证和序列化
- 处理文件上传和多部分请求
- 实现身份验证和授权

### 测试掌握
- 使用TestClient编写全面的API测试
- 测试异步端点和后台任务
- 模拟外部依赖和服务
- 测试数据库操作和事务
- 验证WebSocket通信

## 🔄 渐进式学习路径

### 第一阶段：FastAPI基础(初级)
1. 完成**简单REST API** - 学习基本FastAPI语法
2. 完成**用户认证API** - 掌握安全模式
3. 构建**带验证的Todo API** - 理解Pydantic模型
4. 实现**文件上传API** - 处理多部分请求

### 第二阶段：数据库集成(中级)
1. **带数据库的博客API** - 学习SQLAlchemy与FastAPI
2. **电商产品API** - 处理复杂关系
3. **任务队列API** - 掌握后台处理
4. **实时聊天API** - 实现WebSocket通信

### 第三阶段：企业模式(高级)
1. **微服务订单系统** - 构建分布式系统
2. **事件驱动架构** - 实现CQRS和事件溯源
3. **多租户SaaS API** - 处理复杂授权