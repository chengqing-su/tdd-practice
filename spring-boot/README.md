# Spring Boot TDD Practice

Welcome to Spring Boot Test-Driven Development practice problems! These exercises focus on building web applications, REST APIs, and microservices using Spring Boot with comprehensive testing strategies.

[中文版本](#spring-boot-tdd-实践)

## 🎯 What Makes This Different

Unlike general TDD problems, these exercises specifically target:
- **Spring Boot Framework** patterns and best practices
- **Web application testing** with MockMvc, WebTestClient
- **Database integration testing** with @DataJpaTest, TestContainers
- **REST API development** with proper HTTP status codes and validation
- **Dependency injection testing** with @MockBean and @SpyBean
- **Integration testing** with @SpringBootTest
- **Security testing** with Spring Security Test

## 📊 Difficulty Levels

### 🟢 Beginner
Perfect for developers new to Spring Boot or web development TDD:
- Basic REST controllers with CRUD operations
- Simple data models with JPA
- Basic validation and error handling
- Unit testing with Spring Boot Test
- MockMvc for web layer testing

### 🟡 Intermediate
For developers comfortable with Spring Boot basics:
- Complex business logic services
- Database relationships and transactions
- Security implementation and testing
- External API integration
- Caching and performance optimization
- Custom validation and exception handling

### 🔴 Advanced
Enterprise-level Spring Boot applications:
- Microservices architecture
- Event-driven systems with messaging
- Advanced security with OAuth2/JWT
- Distributed systems testing
- Performance and load testing
- Monitoring and observability

## 🛠 Technology Stack

### Core Technologies
- **Spring Boot 3.x** - Latest version with native compilation support
- **Spring Data JPA** - Database access and ORM
- **Spring Security** - Authentication and authorization
- **Spring Web** - REST API development

### Testing Stack
- **JUnit 5** - Testing framework
- **Mockito** - Mocking framework
- **AssertJ** - Fluent assertions
- **TestContainers** - Integration testing with real databases
- **WireMock** - HTTP service mocking
- **Spring Boot Test** - Spring-specific testing annotations

### Database & Infrastructure
- **H2** - In-memory database for testing
- **PostgreSQL/MySQL** - Production databases
- **Redis** - Caching and session storage
- **Docker** - Containerization for testing

## 🚀 Getting Started

### Prerequisites
- Java 17 or later
- Maven 3.8+ or Gradle 7.5+
- IDE with Spring Boot support (IntelliJ IDEA, Eclipse, VS Code)
- Docker (for integration tests)

### Project Setup
Each problem includes:
1. **Starter template** with Maven/Gradle configuration
2. **Failing tests** to get you started
3. **Detailed requirements** and acceptance criteria
4. **Solution guide** with step-by-step TDD approach
5. **Extension challenges** for further learning

## 📚 Learning Path

1. **Start with Beginner problems** to learn Spring Boot testing fundamentals
2. **Practice web layer testing** with MockMvc and controller tests
3. **Master data layer testing** with @DataJpaTest and repositories
4. **Learn integration testing** with @SpringBootTest
5. **Advance to complex scenarios** with security, messaging, and microservices

## 🧪 TDD Best Practices for Spring Boot

### Test Slice Annotations
- `@WebMvcTest` - Test web layer only
- `@DataJpaTest` - Test JPA repositories
- `@JsonTest` - Test JSON serialization
- `@SpringBootTest` - Full integration tests

### Testing Strategies
- **Unit Tests**: Fast, isolated, mock dependencies
- **Integration Tests**: Test component interactions
- **End-to-End Tests**: Full application testing
- **Contract Tests**: API contract verification

### Performance Considerations
- Use `@MockBean` judiciously to avoid slow context reloads
- Leverage test slices for faster feedback
- Use TestContainers for realistic database testing
- Implement proper test data management

## 📁 Problems Structure

```
spring-boot/
├── beginner/
│   ├── 01-simple-rest-api/
│   ├── 02-book-catalog/
│   ├── 03-user-registration/
│   └── 04-todo-api/
├── intermediate/
│   ├── 01-blog-platform/
│   ├── 02-inventory-system/
│   ├── 03-payment-service/
│   └── 04-notification-system/
└── advanced/
    ├── 01-microservice-communication/
    ├── 02-event-sourcing-api/
    └── 03-real-time-trading/
```

---

# Spring Boot TDD 实践

欢迎来到Spring Boot测试驱动开发实践题！这些练习专注于使用Spring Boot构建Web应用程序、REST API和微服务，并采用全面的测试策略。

## 🎯 有什么不同

与一般的TDD问题不同，这些练习专门针对：
- **Spring Boot框架**模式和最佳实践
- **Web应用程序测试**，使用MockMvc、WebTestClient
- **数据库集成测试**，使用@DataJpaTest、TestContainers
- **REST API开发**，使用正确的HTTP状态码和验证
- **依赖注入测试**，使用@MockBean和@SpyBean
- **集成测试**，使用@SpringBootTest
- **安全测试**，使用Spring Security Test

## 📊 难度级别

### 🟢 初级
非常适合Spring Boot或Web开发TDD新手：
- 带CRUD操作的基本REST控制器
- 带JPA的简单数据模型
- 基本验证和错误处理
- Spring Boot Test单元测试
- Web层的MockMvc测试

### 🟡 中级
适合熟悉Spring Boot基础的开发者：
- 复杂业务逻辑服务
- 数据库关系和事务
- 安全实现和测试
- 外部API集成
- 缓存和性能优化
- 自定义验证和异常处理

### 🔴 高级
企业级Spring Boot应用程序：
- 微服务架构
- 带消息传递的事件驱动系统
- 带OAuth2/JWT的高级安全
- 分布式系统测试
- 性能和负载测试
- 监控和可观测性

## 🛠 技术栈

### 核心技术
- **Spring Boot 3.x** - 支持原生编译的最新版本
- **Spring Data JPA** - 数据库访问和ORM
- **Spring Security** - 身份验证和授权
- **Spring Web** - REST API开发

### 测试技术栈
- **JUnit 5** - 测试框架
- **Mockito** - 模拟框架
- **AssertJ** - 流畅断言
- **TestContainers** - 真实数据库集成测试
- **WireMock** - HTTP服务模拟
- **Spring Boot Test** - Spring特定测试注解

### 数据库和基础设施
- **H2** - 测试用内存数据库
- **PostgreSQL/MySQL** - 生产数据库
- **Redis** - 缓存和会话存储
- **Docker** - 测试容器化

## 🚀 快速开始

### 前置条件
- Java 17或更高版本
- Maven 3.8+或Gradle 7.5+
- 支持Spring Boot的IDE（IntelliJ IDEA、Eclipse、VS Code）
- Docker（用于集成测试）

### 项目设置
每个问题包括：
1. **启动模板**，带Maven/Gradle配置
2. **失败测试**帮你开始
3. **详细需求**和验收标准
4. **解决方案指南**，逐步TDD方法
5. **扩展挑战**，进一步学习

## 📚 学习路径

1. **从初级问题开始**学习Spring Boot测试基础
2. **练习Web层测试**，使用MockMvc和控制器测试
3. **掌握数据层测试**，使用@DataJpaTest和仓库
4. **学习集成测试**，使用@SpringBootTest
5. **进阶到复杂场景**，包括安全、消息传递和微服务

## 🧪 Spring Boot TDD最佳实践

### 测试切片注解
- `@WebMvcTest` - 仅测试Web层
- `@DataJpaTest` - 测试JPA仓库
- `@JsonTest` - 测试JSON序列化
- `@SpringBootTest` - 完整集成测试

### 测试策略
- **单元测试**：快速、隔离、模拟依赖
- **集成测试**：测试组件交互
- **端到端测试**：完整应用程序测试
- **契约测试**：API契约验证

### 性能考虑
- 谨慎使用`@MockBean`避免缓慢的上下文重新加载
- 利用测试切片获得更快反馈
- 使用TestContainers进行现实数据库测试
- 实施适当的测试数据管理

## 📁 问题结构

```
spring-boot/
├── beginner/
│   ├── 01-simple-rest-api/
│   ├── 02-book-catalog/
│   ├── 03-user-registration/
│   └── 04-todo-api/
├── intermediate/
│   ├── 01-blog-platform/
│   ├── 02-inventory-system/
│   ├── 03-payment-service/
│   └── 04-notification-system/
└── advanced/
    ├── 01-microservice-communication/
    ├── 02-event-sourcing-api/
    └── 03-real-time-trading/
```