# Simple REST API

## Problem Description

Create a simple REST API for managing a collection of books using Spring Boot. This is your first step into web development TDD with proper HTTP methods, status codes, and JSON handling.

## Requirements

### Book Entity
- `Book` - Simple POJO with id, title, author, isbn, publicationYear
- Basic validation annotations
- Proper JSON serialization

### BookController
- `GET /api/books` - Get all books
- `GET /api/books/{id}` - Get book by ID
- `POST /api/books` - Create new book
- `PUT /api/books/{id}` - Update existing book
- `DELETE /api/books/{id}` - Delete book

### Response Format
- Success responses with appropriate HTTP status codes
- Error responses with proper error messages
- JSON content type for all responses

### Validation
- Required fields: title, author, isbn
- ISBN format validation (basic pattern)
- Publication year range validation (1800-current year)

## Project Setup

### Maven Dependencies (pom.xml)
```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-validation</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>
```

## Expected API Behavior

### GET /api/books
```bash
# Request
curl -X GET http://localhost:8080/api/books

# Response: 200 OK
[
  {
    "id": 1,
    "title": "Clean Code",
    "author": "Robert Martin",
    "isbn": "978-0132350884",
    "publicationYear": 2008
  }
]
```

### POST /api/books
```bash
# Request
curl -X POST http://localhost:8080/api/books \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Clean Code",
    "author": "Robert Martin",
    "isbn": "978-0132350884",
    "publicationYear": 2008
  }'

# Response: 201 Created
{
  "id": 1,
  "title": "Clean Code",
  "author": "Robert Martin",
  "isbn": "978-0132350884",
  "publicationYear": 2008
}
```

### Error Cases
```bash
# Invalid data
curl -X POST http://localhost:8080/api/books \
  -H "Content-Type: application/json" \
  -d '{"title": ""}'

# Response: 400 Bad Request
{
  "message": "Validation failed",
  "errors": [
    "Title is required",
    "Author is required",
    "ISBN is required"
  ]
}
```

## TDD Steps

### 1. Start with Controller Tests
```java
@WebMvcTest(BookController.class)
class BookControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private BookService bookService;

    @Test
    void shouldReturnAllBooks() throws Exception {
        // Given
        List<Book> books = Arrays.asList(
            new Book(1L, "Clean Code", "Robert Martin", "978-0132350884", 2008)
        );
        when(bookService.getAllBooks()).thenReturn(books);

        // When & Then
        mockMvc.perform(get("/api/books"))
               .andExpect(status().isOk())
               .andExpect(jsonPath("$", hasSize(1)))
               .andExpect(jsonPath("$[0].title").value("Clean Code"));
    }
}
```

### 2. TDD Process
1. **Red**: Write failing controller test
2. **Green**: Create minimal controller to pass test
3. **Red**: Add service layer test
4. **Green**: Implement service
5. **Red**: Add validation tests
6. **Green**: Add validation annotations
7. **Refactor**: Clean up code and tests

## Learning Goals

- Spring Boot web layer testing with `@WebMvcTest`
- MockMvc for HTTP request simulation
- JSON serialization/deserialization
- Bean validation with annotations
- HTTP status codes and RESTful principles
- Mocking services with `@MockBean`
- AssertJ and JSONPath assertions

## Testing Checklist

### Controller Layer Tests
- [ ] GET all books returns 200 and book list
- [ ] GET book by ID returns 200 and correct book
- [ ] GET non-existent book returns 404
- [ ] POST valid book returns 201 and created book
- [ ] POST invalid book returns 400 with error details
- [ ] PUT existing book returns 200 and updated book
- [ ] PUT non-existent book returns 404
- [ ] DELETE existing book returns 204
- [ ] DELETE non-existent book returns 404

### Validation Tests
- [ ] Empty title rejected
- [ ] Empty author rejected
- [ ] Invalid ISBN format rejected
- [ ] Invalid publication year rejected
- [ ] Valid book accepted

### JSON Serialization Tests
- [ ] Book serializes to correct JSON structure
- [ ] JSON deserializes to correct Book object
- [ ] Error responses have proper JSON format

## Extension Ideas

1. **Add pagination** - Implement pageable responses
2. **Add search** - Filter books by title, author
3. **Add sorting** - Sort books by different fields
4. **Custom exceptions** - Create specific exception types
5. **API documentation** - Add Swagger/OpenAPI
6. **Response DTOs** - Separate internal models from API responses

---

## 问题描述

使用Spring Boot创建管理书籍集合的简单REST API。这是你进入Web开发TDD的第一步，包括适当的HTTP方法、状态码和JSON处理。

## 需求

### Book实体
- `Book` - 包含id、title、author、isbn、publicationYear的简单POJO
- 基本验证注解
- 适当的JSON序列化

### BookController
- `GET /api/books` - 获取所有书籍
- `GET /api/books/{id}` - 根据ID获取书籍
- `POST /api/books` - 创建新书籍
- `PUT /api/books/{id}` - 更新现有书籍
- `DELETE /api/books/{id}` - 删除书籍

### 响应格式
- 带适当HTTP状态码的成功响应
- 带适当错误信息的错误响应
- 所有响应的JSON内容类型

### 验证
- 必填字段：title、author、isbn
- ISBN格式验证（基本模式）
- 出版年份范围验证（1800-当前年份）

## 项目设置

### Maven依赖 (pom.xml)
```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-validation</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>
```

## 预期API行为

### GET /api/books
```bash
# 请求
curl -X GET http://localhost:8080/api/books

# 响应: 200 OK
[
  {
    "id": 1,
    "title": "代码整洁之道",
    "author": "Robert Martin",
    "isbn": "978-0132350884",
    "publicationYear": 2008
  }
]
```

### POST /api/books
```bash
# 请求
curl -X POST http://localhost:8080/api/books \
  -H "Content-Type: application/json" \
  -d '{
    "title": "代码整洁之道",
    "author": "Robert Martin",
    "isbn": "978-0132350884",
    "publicationYear": 2008
  }'

# 响应: 201 Created
{
  "id": 1,
  "title": "代码整洁之道",
  "author": "Robert Martin",
  "isbn": "978-0132350884",
  "publicationYear": 2008
}
```

### 错误情况
```bash
# 无效数据
curl -X POST http://localhost:8080/api/books \
  -H "Content-Type: application/json" \
  -d '{"title": ""}'

# 响应: 400 Bad Request
{
  "message": "验证失败",
  "errors": [
    "标题是必需的",
    "作者是必需的",
    "ISBN是必需的"
  ]
}
```

## TDD 步骤

### 1. 从控制器测试开始
```java
@WebMvcTest(BookController.class)
class BookControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private BookService bookService;

    @Test
    void shouldReturnAllBooks() throws Exception {
        // Given
        List<Book> books = Arrays.asList(
            new Book(1L, "代码整洁之道", "Robert Martin", "978-0132350884", 2008)
        );
        when(bookService.getAllBooks()).thenReturn(books);

        // When & Then
        mockMvc.perform(get("/api/books"))
               .andExpect(status().isOk())
               .andExpected(jsonPath("$", hasSize(1)))
               .andExpect(jsonPath("$[0].title").value("代码整洁之道"));
    }
}
```

### 2. TDD 过程
1. **红灯**: 编写失败的控制器测试
2. **绿灯**: 创建通过测试的最小控制器
3. **红灯**: 添加服务层测试
4. **绿灯**: 实现服务
5. **红灯**: 添加验证测试
6. **绿灯**: 添加验证注解
7. **重构**: 清理代码和测试

## 学习目标

- 使用`@WebMvcTest`的Spring Boot Web层测试
- HTTP请求模拟的MockMvc
- JSON序列化/反序列化
- 注解Bean验证
- HTTP状态码和RESTful原则
- 使用`@MockBean`模拟服务
- AssertJ和JSONPath断言

## 测试清单

### 控制器层测试
- [ ] GET所有书籍返回200和书籍列表
- [ ] 根据ID GET书籍返回200和正确书籍
- [ ] GET不存在的书籍返回404
- [ ] POST有效书籍返回201和创建的书籍
- [ ] POST无效书籍返回400和错误详情
- [ ] PUT现有书籍返回200和更新的书籍
- [ ] PUT不存在的书籍返回404
- [ ] DELETE现有书籍返回204
- [ ] DELETE不存在的书籍返回404

### 验证测试
- [ ] 空标题被拒绝
- [ ] 空作者被拒绝
- [ ] 无效ISBN格式被拒绝
- [ ] 无效出版年份被拒绝
- [ ] 有效书籍被接受

### JSON序列化测试
- [ ] Book序列化为正确的JSON结构
- [ ] JSON反序列化为正确的Book对象
- [ ] 错误响应有适当的JSON格式

## 扩展想法

1. **添加分页** - 实现可分页响应
2. **添加搜索** - 按标题、作者过滤书籍
3. **添加排序** - 按不同字段排序书籍
4. **自定义异常** - 创建特定异常类型
5. **API文档** - 添加Swagger/OpenAPI
6. **响应DTO** - 将内部模型与API响应分离