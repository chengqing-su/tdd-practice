# FastAPI Beginner Problems

These beginner problems introduce the fundamentals of building and testing FastAPI applications using Test-Driven Development. Perfect for developers new to FastAPI or API testing.

[中文版本](#fastapi-初级问题)

## Problems

1. **[Simple REST API](01-simple-rest-api.md)** - Basic CRUD operations with in-memory storage
2. **[User Authentication API](02-user-auth-api.md)** - JWT authentication and user management
3. **[Todo API with Validation](03-todo-api-validation.md)** - Pydantic models and request validation
4. **[File Upload API](04-file-upload-api.md)** - File handling and multipart requests

## What You'll Learn

### FastAPI Fundamentals
- **FastAPI Application Structure** - Creating and organizing FastAPI apps
- **Request/Response Models** - Using Pydantic for data validation
- **HTTP Methods** - Implementing GET, POST, PUT, DELETE endpoints
- **Status Codes** - Returning appropriate HTTP status codes
- **Error Handling** - Creating custom exception handlers

### Testing with FastAPI
- **TestClient** - FastAPI's built-in testing client
- **API Testing Patterns** - Testing endpoints, status codes, and responses
- **Test Fixtures** - Setting up test data and cleanup
- **Assertion Techniques** - Verifying API behavior
- **Test Organization** - Structuring tests for maintainability

### API Design Principles
- **RESTful Design** - Following REST conventions
- **Request Validation** - Input validation with Pydantic
- **Response Formatting** - Consistent API responses
- **Documentation** - Automatic OpenAPI/Swagger docs
- **Error Messages** - Clear, actionable error responses

## Prerequisites

- Basic Python knowledge (functions, classes, dictionaries)
- Understanding of HTTP concepts (methods, status codes)
- Basic testing concepts (assertions, test functions)
- Familiarity with JSON data format

## Problem Complexity

### Simple REST API
**Complexity**: ⭐⭐
**Focus Areas**:
- Basic FastAPI app setup
- CRUD operations (Create, Read, Update, Delete)
- In-memory data storage
- HTTP status codes
- JSON request/response handling

**Key Learning Points**:
- FastAPI application structure
- Route definition with decorators
- Request body parsing
- Response model definition

### User Authentication API
**Complexity**: ⭐⭐⭐
**Focus Areas**:
- User registration and login
- JWT token generation and validation
- Password hashing and verification
- Protected routes with authentication
- Custom middleware and dependencies

**Key Learning Points**:
- Authentication flow implementation
- Security best practices
- Dependency injection in FastAPI
- Custom exception handling

### Todo API with Validation
**Complexity**: ⭐⭐⭐
**Focus Areas**:
- Advanced Pydantic models
- Field validation and custom validators
- Error response formatting
- Data transformation
- API versioning concepts

**Key Learning Points**:
- Complex validation rules
- Custom error messages
- Request/response model separation
- Input sanitization

### File Upload API
**Complexity**: ⭐⭐⭐
**Focus Areas**:
- Multipart form data handling
- File validation (size, type)
- File storage and retrieval
- Progress tracking
- Error handling for large files

**Key Learning Points**:
- File handling in web APIs
- Multipart request processing
- File system operations
- Async file operations

---

# FastAPI 初级问题

这些初级问题使用测试驱动开发介绍构建和测试FastAPI应用程序的基础知识。非常适合FastAPI或API测试的新手开发者。

## 问题列表

1. **[简单REST API](01-simple-rest-api.md)** - 内存存储的基本CRUD操作
2. **[用户认证API](02-user-auth-api.md)** - JWT认证和用户管理
3. **[带验证的Todo API](03-todo-api-validation.md)** - Pydantic模型和请求验证
4. **[文件上传API](04-file-upload-api.md)** - 文件处理和多部分请求

## 你将学到什么

### FastAPI基础
- **FastAPI应用结构** - 创建和组织FastAPI应用
- **请求/响应模型** - 使用Pydantic进行数据验证
- **HTTP方法** - 实现GET、POST、PUT、DELETE端点
- **状态码** - 返回适当的HTTP状态码
- **错误处理** - 创建自定义异常处理器

### FastAPI测试
- **TestClient** - FastAPI内置测试客户端
- **API测试模式** - 测试端点、状态码和响应
- **测试夹具** - 设置测试数据和清理
- **断言技术** - 验证API行为
- **测试组织** - 构建可维护的测试

## 前置条件

- 基本Python知识（函数、类、字典）
- 理解HTTP概念（方法、状态码）
- 基本测试概念（断言、测试函数）
- 熟悉JSON数据格式