# FastAPI Intermediate Problems

These intermediate problems focus on database integration, advanced FastAPI features, and real-world application patterns. Perfect for developers ready to build production-grade APIs.

[中文版本](#fastapi-中级问题)

## Problems

1. **[Blog API with Database](01-blog-api-database.md)** - SQLAlchemy integration, complex relationships, and database testing
2. **[E-commerce Product API](02-ecommerce-product-api.md)** - Complex business logic, inventory management, and transactions
3. **[Task Queue API](03-task-queue-api.md)** - Background tasks, async processing, and job management
4. **[Real-time Chat API](04-realtime-chat-api.md)** - WebSockets, real-time communication, and message handling

## What You'll Learn

### Database Integration
- **SQLAlchemy ORM** - Database models, relationships, and queries
- **Database Testing** - Test databases, fixtures, and transaction management
- **Migrations** - Alembic integration for schema changes
- **Connection Pooling** - Database connection management
- **Query Optimization** - N+1 problem prevention and performance

### Advanced FastAPI Features
- **Background Tasks** - Async task processing and job queues
- **WebSockets** - Real-time bidirectional communication
- **Middleware** - Custom middleware for cross-cutting concerns
- **Dependencies** - Advanced dependency injection patterns
- **Event Handlers** - Startup and shutdown event handling

### Testing Patterns
- **Database Testing** - Isolated test databases and cleanup
- **Async Testing** - Testing async endpoints and operations
- **Integration Testing** - End-to-end testing with external dependencies
- **Performance Testing** - Load testing and benchmarking
- **WebSocket Testing** - Real-time communication testing

### Production Patterns
- **Error Handling** - Comprehensive error management
- **Logging** - Structured logging and monitoring
- **Configuration** - Environment-based configuration
- **Security** - Advanced authentication and authorization
- **Caching** - Response caching and data caching

## Prerequisites

- Completion of [Beginner Problems](../beginner/)
- Database concepts (SQL, relationships, transactions)
- Understanding of async/await in Python
- Basic knowledge of testing patterns
- Familiarity with REST API principles

## Problem Complexity

### Blog API with Database
**Complexity**: ⭐⭐⭐⭐
**Focus Areas**:
- SQLAlchemy ORM integration
- Complex entity relationships (users, posts, comments, tags)
- Database testing with pytest
- Query optimization and N+1 prevention
- Database transactions and rollbacks

**Key Learning Points**:
- Database model design
- Relationship handling in ORM
- Database testing strategies
- Performance considerations

### E-commerce Product API
**Complexity**: ⭐⭐⭐⭐
**Focus Areas**:
- Complex business logic (inventory, pricing, discounts)
- Multi-table transactions
- Inventory management and concurrency
- Price calculation and validation
- Order processing workflows

**Key Learning Points**:
- Business logic implementation
- Transaction management
- Concurrency handling
- Complex validation rules

### Task Queue API
**Complexity**: ⭐⭐⭐⭐⭐
**Focus Areas**:
- Background task processing
- Job queue management
- Async operation patterns
- Task monitoring and status tracking
- Error handling and retries

**Key Learning Points**:
- Async programming patterns
- Background job processing
- Task queue architecture
- Monitoring and observability

### Real-time Chat API
**Complexity**: ⭐⭐⭐⭐⭐
**Focus Areas**:
- WebSocket implementation
- Real-time message handling
- Connection management
- Message persistence and history
- Scaling real-time applications

**Key Learning Points**:
- WebSocket protocols
- Real-time architecture
- Connection state management
- Message broadcasting

---

# FastAPI 中级问题

这些中级问题专注于数据库集成、高级FastAPI功能和真实世界应用模式。非常适合准备构建生产级API的开发者。

## 问题列表

1. **[带数据库的博客API](01-blog-api-database.md)** - SQLAlchemy集成、复杂关系和数据库测试
2. **[电商产品API](02-ecommerce-product-api.md)** - 复杂业务逻辑、库存管理和事务
3. **[任务队列API](03-task-queue-api.md)** - 后台任务、异步处理和作业管理
4. **[实时聊天API](04-realtime-chat-api.md)** - WebSocket、实时通信和消息处理

## 你将学到什么

### 数据库集成
- **SQLAlchemy ORM** - 数据库模型、关系和查询
- **数据库测试** - 测试数据库、夹具和事务管理
- **迁移** - 模式更改的Alembic集成
- **连接池** - 数据库连接管理
- **查询优化** - N+1问题预防和性能

### 高级FastAPI功能
- **后台任务** - 异步任务处理和作业队列
- **WebSocket** - 实时双向通信
- **中间件** - 横切关注点的自定义中间件
- **依赖项** - 高级依赖注入模式
- **事件处理器** - 启动和关闭事件处理

## 前置条件

- 完成[初级问题](../beginner/)
- 数据库概念（SQL、关系、事务）
- 理解Python中的async/await
- 测试模式的基本知识
- 熟悉REST API原则