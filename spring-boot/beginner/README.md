# Spring Boot Beginner Problems

Welcome to Spring Boot TDD beginner problems! These exercises will teach you the fundamentals of testing web applications, REST APIs, and database integration using Spring Boot's testing framework.

[中文版本](#spring-boot-初级问题)

## Problems

1. **[Simple REST API](01-simple-rest-api.md)** - Basic CRUD operations, MockMvc testing, JSON handling
2. **[Book Catalog with Database](02-book-catalog-with-database.md)** - JPA integration, repository testing, data layer TDD
3. **[User Registration System](03-user-registration-system.md)** - Spring Security, validation, password encoding
4. **[Todo API with Validation](04-todo-api-with-validation.md)** - Advanced validation, business rules, user associations

## What You'll Learn

### Spring Boot Testing Fundamentals
- `@SpringBootTest` for integration testing
- `@WebMvcTest` for web layer testing
- `@DataJpaTest` for repository testing
- Test slicing for focused testing

### Web Layer Testing
- MockMvc for HTTP request simulation
- JSON serialization/deserialization testing
- HTTP status code validation
- Request/response body testing

### Data Layer Testing
- JPA repository testing with TestEntityManager
- Custom query method testing
- Database constraint validation
- Transaction testing

### Security Testing
- Spring Security Test integration
- `@WithMockUser` for authentication testing
- Password encoding testing
- Authorization testing

### Validation and Error Handling
- Bean Validation with annotations
- Custom validators
- Exception handling testing
- Error response formatting

## Prerequisites

- Basic Java knowledge
- Understanding of REST API concepts
- Familiarity with Maven/Gradle
- Basic Spring framework knowledge (helpful but not required)

## Getting Started

### Step 1: Environment Setup
1. Install Java 17 or later
2. Install Maven 3.8+ or Gradle 7.5+
3. Choose an IDE (IntelliJ IDEA recommended)
4. Install Docker (for advanced integration testing)

### Step 2: Start with Problem 1
1. Read the problem description carefully
2. Set up the initial Spring Boot project
3. Write your first failing test
4. Follow the TDD cycle: Red → Green → Refactor

### Step 3: Progressive Learning
- Complete problems in order
- Focus on understanding test structure
- Practice the TDD cycle consistently
- Run tests frequently during development

## Spring Boot Testing Patterns

### Controller Testing Pattern
```java
@WebMvcTest(BookController.class)
class BookControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private BookService bookService;

    @Test
    void shouldReturnAllBooks() throws Exception {
        // Arrange
        List<Book> books = Arrays.asList(createBook());
        when(bookService.getAllBooks()).thenReturn(books);

        // Act & Assert
        mockMvc.perform(get("/api/books"))
               .andExpect(status().isOk())
               .andExpected(jsonPath("$", hasSize(1)));
    }
}
```

### Repository Testing Pattern
```java
@DataJpaTest
class BookRepositoryTest {

    @Autowired
    private TestEntityManager entityManager;

    @Autowired
    private BookRepository bookRepository;

    @Test
    void shouldFindBooksByAuthor() {
        // Arrange
        Book book = createBook();
        entityManager.persistAndFlush(book);

        // Act
        List<Book> result = bookRepository.findByAuthor("Test Author");

        // Assert
        assertThat(result).hasSize(1);
    }
}
```

### Service Testing Pattern
```java
@ExtendWith(MockitoExtension.class)
class BookServiceTest {

    @Mock
    private BookRepository bookRepository;

    @InjectMocks
    private BookService bookService;

    @Test
    void shouldCreateBook() {
        // Arrange
        Book book = createBook();
        when(bookRepository.save(any(Book.class))).thenReturn(book);

        // Act
        Book result = bookService.createBook(createBookRequest());

        // Assert
        assertThat(result).isNotNull();
        verify(bookRepository).save(any(Book.class));
    }
}
```

## Common Testing Annotations

### Test Slicing Annotations
- `@WebMvcTest` - Load only web layer components
- `@DataJpaTest` - Load only JPA repositories and entities
- `@JsonTest` - Test JSON serialization/deserialization
- `@SpringBootTest` - Load full application context

### Mocking Annotations
- `@MockBean` - Create and inject mocks in Spring context
- `@SpyBean` - Create and inject spies in Spring context
- `@Mock` - Create mock objects (Mockito)
- `@InjectMocks` - Inject mocks into test subject

### Security Testing
- `@WithMockUser` - Run test with authenticated user
- `@WithUserDetails` - Load user from UserDetailsService
- `@WithAnonymousUser` - Run test as anonymous user

## Best Practices

### Test Organization
1. **Arrange-Act-Assert (AAA)** pattern
2. **Given-When-Then** pattern for BDD style
3. Use descriptive test method names
4. Keep tests focused and independent

### Test Data Management
1. Use test data builders or factory methods
2. Clean up data between tests when needed
3. Use `@Sql` for complex test data setup
4. Prefer in-memory databases for testing

### Performance Considerations
1. Use appropriate test slices to load only needed components
2. Avoid `@SpringBootTest` for unit tests
3. Use `@MockBean` sparingly to prevent context caching issues
4. Consider TestContainers for realistic integration tests

### Assertion Libraries
- **AssertJ** - Fluent assertions with better error messages
- **Hamcrest** - Matcher-based assertions
- **JSONPath** - JSON content assertions
- **Spring Test** - Web-specific assertions

## Common Pitfalls to Avoid

1. **Over-mocking** - Don't mock everything; test real interactions when possible
2. **Context loading abuse** - Don't use `@SpringBootTest` for simple unit tests
3. **Test data pollution** - Clean up shared state between tests
4. **Ignoring test performance** - Keep test execution time reasonable
5. **Testing implementation details** - Focus on behavior, not implementation

## Next Steps

After completing beginner problems:
1. Move to [Intermediate Problems](../intermediate/) for complex business logic
2. Learn about microservices testing patterns
3. Explore performance testing with Spring Boot
4. Study security testing in depth
5. Practice with real-world scenarios

---

# Spring Boot 初级问题

欢迎来到Spring Boot TDD初级问题！这些练习将教你使用Spring Boot测试框架测试Web应用程序、REST API和数据库集成的基础知识。

## 问题列表

1. **[简单REST API](01-simple-rest-api.md)** - 基本CRUD操作、MockMvc测试、JSON处理
2. **[带数据库的书籍目录](02-book-catalog-with-database.md)** - JPA集成、仓库测试、数据层TDD
3. **[用户注册系统](03-user-registration-system.md)** - Spring Security、验证、密码编码
4. **[带验证的Todo API](04-todo-api-with-validation.md)** - 高级验证、业务规则、用户关联

## 你将学到什么

### Spring Boot测试基础
- 集成测试的`@SpringBootTest`
- Web层测试的`@WebMvcTest`
- 仓库测试的`@DataJpaTest`
- 专注测试的测试切片

### Web层测试
- HTTP请求模拟的MockMvc
- JSON序列化/反序列化测试
- HTTP状态码验证
- 请求/响应体测试

### 数据层测试
- 使用TestEntityManager的JPA仓库测试
- 自定义查询方法测试
- 数据库约束验证
- 事务测试

### 安全测试
- Spring Security Test集成
- 身份验证测试的`@WithMockUser`
- 密码编码测试
- 授权测试

### 验证和错误处理
- 注解Bean验证
- 自定义验证器
- 异常处理测试
- 错误响应格式化

## 前置条件

- 基本Java知识
- 理解REST API概念
- 熟悉Maven/Gradle
- 基本Spring框架知识（有帮助但非必需）

## 快速开始

### 步骤1：环境设置
1. 安装Java 17或更高版本
2. 安装Maven 3.8+或Gradle 7.5+
3. 选择IDE（推荐IntelliJ IDEA）
4. 安装Docker（用于高级集成测试）

### 步骤2：从问题1开始
1. 仔细阅读问题描述
2. 设置初始Spring Boot项目
3. 编写第一个失败测试
4. 遵循TDD循环：红灯→绿灯→重构

### 步骤3：渐进学习
- 按顺序完成问题
- 专注理解测试结构
- 始终如一地实践TDD循环
- 开发过程中频繁运行测试

## 最佳实践

### 测试组织
1. **排列-执行-断言(AAA)**模式
2. BDD风格的**给定-当-那么**模式
3. 使用描述性测试方法名
4. 保持测试专注和独立

### 测试数据管理
1. 使用测试数据构建器或工厂方法
2. 需要时在测试间清理数据
3. 使用`@Sql`进行复杂测试数据设置
4. 测试优选内存数据库

### 性能考虑
1. 使用适当的测试切片仅加载需要的组件
2. 单元测试避免`@SpringBootTest`
3. 谨慎使用`@MockBean`以防止上下文缓存问题
4. 考虑TestContainers进行现实集成测试

## 下一步

完成初级问题后：
1. 转向[中级问题](../intermediate/)学习复杂业务逻辑
2. 学习微服务测试模式
3. 探索Spring Boot性能测试
4. 深入研究安全测试
5. 用真实场景练习