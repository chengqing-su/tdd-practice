# Book Catalog with Database

## Problem Description

Extend the simple REST API to use a real database with Spring Data JPA. Learn how to test data layer components and integrate database operations with your web layer using TDD.

## Requirements

### Data Model
- `Book` entity with JPA annotations
- Auto-generated ID with `@GeneratedValue`
- Proper column mappings and constraints
- Database table creation via schema.sql or JPA

### Repository Layer
- `BookRepository` extending `JpaRepository<Book, Long>`
- Custom query methods for finding books by various criteria
- Derived queries for common search operations

### Service Layer
- `BookService` with business logic
- Repository integration
- Exception handling for not found cases
- Transaction management

### Database Configuration
- H2 in-memory database for testing
- Separate test properties configuration
- Database initialization and cleanup

## Project Setup

### Additional Dependencies
```xml
<dependencies>
    <!-- Previous dependencies... -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>
    <dependency>
        <groupId>com.h2database</groupId>
        <artifactId>h2</artifactId>
        <scope>runtime</scope>
    </dependency>
    <dependency>
        <groupId>org.testcontainers</groupId>
        <artifactId>junit-jupiter</artifactId>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>org.testcontainers</groupId>
        <artifactId>postgresql</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>
```

### Application Properties
```properties
# application-test.properties
spring.datasource.url=jdbc:h2:mem:testdb
spring.datasource.driver-class-name=org.h2.Driver
spring.jpa.hibernate.ddl-auto=create-drop
spring.jpa.show-sql=true
logging.level.org.springframework.orm.jpa=DEBUG
```

## TDD Testing Layers

### 1. Repository Layer Testing
```java
@DataJpaTest
class BookRepositoryTest {

    @Autowired
    private TestEntityManager entityManager;

    @Autowired
    private BookRepository bookRepository;

    @Test
    void shouldFindBooksByAuthor() {
        // Given
        Book book = new Book();
        book.setTitle("Clean Code");
        book.setAuthor("Robert Martin");
        book.setIsbn("978-0132350884");
        book.setPublicationYear(2008);
        entityManager.persistAndFlush(book);

        // When
        List<Book> foundBooks = bookRepository.findByAuthor("Robert Martin");

        // Then
        assertThat(foundBooks).hasSize(1);
        assertThat(foundBooks.get(0).getTitle()).isEqualTo("Clean Code");
    }

    @Test
    void shouldFindBooksByTitleContainingIgnoreCase() {
        // Given
        Book book = new Book();
        book.setTitle("Clean Code");
        book.setAuthor("Robert Martin");
        book.setIsbn("978-0132350884");
        book.setPublicationYear(2008);
        entityManager.persistAndFlush(book);

        // When
        List<Book> foundBooks = bookRepository.findByTitleContainingIgnoreCase("clean");

        // Then
        assertThat(foundBooks).hasSize(1);
        assertThat(foundBooks.get(0).getTitle()).isEqualTo("Clean Code");
    }
}
```

### 2. Service Layer Testing
```java
@ExtendWith(MockitoExtension.class)
class BookServiceTest {

    @Mock
    private BookRepository bookRepository;

    @InjectMocks
    private BookService bookService;

    @Test
    void shouldReturnAllBooks() {
        // Given
        List<Book> books = Arrays.asList(createBook());
        when(bookRepository.findAll()).thenReturn(books);

        // When
        List<Book> result = bookService.getAllBooks();

        // Then
        assertThat(result).hasSize(1);
        verify(bookRepository).findAll();
    }

    @Test
    void shouldThrowExceptionWhenBookNotFound() {
        // Given
        when(bookRepository.findById(1L)).thenReturn(Optional.empty());

        // When & Then
        assertThatThrownBy(() -> bookService.getBookById(1L))
            .isInstanceOf(BookNotFoundException.class)
            .hasMessage("Book not found with id: 1");
    }

    private Book createBook() {
        Book book = new Book();
        book.setId(1L);
        book.setTitle("Clean Code");
        book.setAuthor("Robert Martin");
        book.setIsbn("978-0132350884");
        book.setPublicationYear(2008);
        return book;
    }
}
```

### 3. Integration Testing
```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@ActiveProfiles("test")
class BookIntegrationTest {

    @Autowired
    private TestRestTemplate restTemplate;

    @Autowired
    private BookRepository bookRepository;

    @BeforeEach
    void setUp() {
        bookRepository.deleteAll();
    }

    @Test
    void shouldCreateAndRetrieveBook() {
        // Given
        Book book = new Book();
        book.setTitle("Clean Code");
        book.setAuthor("Robert Martin");
        book.setIsbn("978-0132350884");
        book.setPublicationYear(2008);

        // When - Create book
        ResponseEntity<Book> createResponse = restTemplate.postForEntity(
            "/api/books", book, Book.class);

        // Then - Verify creation
        assertThat(createResponse.getStatusCode()).isEqualTo(HttpStatus.CREATED);
        assertThat(createResponse.getBody().getId()).isNotNull();

        // When - Retrieve book
        ResponseEntity<Book[]> getResponse = restTemplate.getForEntity(
            "/api/books", Book[].class);

        // Then - Verify retrieval
        assertThat(getResponse.getStatusCode()).isEqualTo(HttpStatus.OK);
        assertThat(getResponse.getBody()).hasSize(1);
        assertThat(getResponse.getBody()[0].getTitle()).isEqualTo("Clean Code");
    }
}
```

## Custom Repository Methods

```java
public interface BookRepository extends JpaRepository<Book, Long> {

    List<Book> findByAuthor(String author);

    List<Book> findByTitleContainingIgnoreCase(String title);

    List<Book> findByPublicationYearBetween(Integer startYear, Integer endYear);

    @Query("SELECT b FROM Book b WHERE b.author = ?1 AND b.publicationYear > ?2")
    List<Book> findBooksByAuthorAndYearAfter(String author, Integer year);

    @Modifying
    @Query("DELETE FROM Book b WHERE b.publicationYear < ?1")
    int deleteOldBooks(Integer year);
}
```

## Exception Handling

```java
@ControllerAdvice
public class BookExceptionHandler {

    @ExceptionHandler(BookNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleBookNotFoundException(BookNotFoundException ex) {
        ErrorResponse error = new ErrorResponse("BOOK_NOT_FOUND", ex.getMessage());
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(error);
    }

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ErrorResponse> handleValidationException(MethodArgumentNotValidException ex) {
        List<String> errors = ex.getBindingResult().getFieldErrors().stream()
            .map(DefaultMessageSourceResolvable::getDefaultMessage)
            .collect(Collectors.toList());

        ErrorResponse error = new ErrorResponse("VALIDATION_FAILED", "Invalid input", errors);
        return ResponseEntity.status(HttpStatus.BAD_REQUEST).body(error);
    }
}
```

## TDD Steps

1. **Red**: Write repository test for basic CRUD operations
2. **Green**: Create Book entity with JPA annotations
3. **Red**: Write repository test for custom query methods
4. **Green**: Add custom methods to BookRepository
5. **Red**: Write service test for business logic
6. **Green**: Implement BookService with repository integration
7. **Red**: Write controller test with mocked service
8. **Green**: Update controller to use service
9. **Red**: Write integration test for full flow
10. **Green**: Ensure all layers work together
11. **Refactor**: Clean up code and optimize queries

## Learning Goals

- Spring Data JPA repository testing with `@DataJpaTest`
- TestEntityManager for test data setup
- Custom repository query methods
- Service layer testing with mocked repositories
- Integration testing with `@SpringBootTest`
- Database transaction management
- Exception handling and custom exceptions
- Test property configuration

## Testing Checklist

### Repository Tests
- [ ] Basic CRUD operations work correctly
- [ ] Custom query methods return expected results
- [ ] Empty results handled properly
- [ ] Query parameters work correctly
- [ ] Database constraints are enforced

### Service Tests
- [ ] Business logic executes correctly
- [ ] Repository methods are called appropriately
- [ ] Exceptions are thrown for invalid scenarios
- [ ] Transactions are handled properly
- [ ] Edge cases are covered

### Integration Tests
- [ ] Full request-response cycle works
- [ ] Database operations persist correctly
- [ ] HTTP status codes are correct
- [ ] Error responses are properly formatted
- [ ] Multiple requests work independently

## Advanced Testing with TestContainers

```java
@SpringBootTest
@Testcontainers
class BookIntegrationTestWithPostgreSQL {

    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:13")
            .withDatabaseName("testdb")
            .withUsername("test")
            .withPassword("test");

    @DynamicPropertySource
    static void configureProperties(DynamicPropertyRegistry registry) {
        registry.add("spring.datasource.url", postgres::getJdbcUrl);
        registry.add("spring.datasource.username", postgres::getUsername);
        registry.add("spring.datasource.password", postgres::getPassword);
    }

    @Test
    void shouldWorkWithRealPostgreSQL() {
        // Test with actual PostgreSQL instance
    }
}
```

---

## 问题描述

扩展简单REST API以使用Spring Data JPA的真实数据库。学习如何测试数据层组件，并使用TDD将数据库操作与Web层集成。

## 需求

### 数据模型
- 带JPA注解的`Book`实体
- 使用`@GeneratedValue`的自动生成ID
- 适当的列映射和约束
- 通过schema.sql或JPA创建数据库表

### 仓库层
- 扩展`JpaRepository<Book, Long>`的`BookRepository`
- 根据各种条件查找书籍的自定义查询方法
- 常见搜索操作的派生查询

### 服务层
- 带业务逻辑的`BookService`
- 仓库集成
- 未找到情况的异常处理
- 事务管理

### 数据库配置
- 测试用H2内存数据库
- 独立的测试属性配置
- 数据库初始化和清理

## 项目设置

### 额外依赖
```xml
<dependencies>
    <!-- 之前的依赖... -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>
    <dependency>
        <groupId>com.h2database</groupId>
        <artifactId>h2</artifactId>
        <scope>runtime</scope>
    </dependency>
    <dependency>
        <groupId>org.testcontainers</groupId>
        <artifactId>junit-jupiter</artifactId>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>org.testcontainers</groupId>
        <artifactId>postgresql</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>
```

### 应用属性
```properties
# application-test.properties
spring.datasource.url=jdbc:h2:mem:testdb
spring.datasource.driver-class-name=org.h2.Driver
spring.jpa.hibernate.ddl-auto=create-drop
spring.jpa.show-sql=true
logging.level.org.springframework.orm.jpa=DEBUG
```

## TDD 测试层

### 1. 仓库层测试
```java
@DataJpaTest
class BookRepositoryTest {

    @Autowired
    private TestEntityManager entityManager;

    @Autowired
    private BookRepository bookRepository;

    @Test
    void shouldFindBooksByAuthor() {
        // Given
        Book book = new Book();
        book.setTitle("代码整洁之道");
        book.setAuthor("Robert Martin");
        book.setIsbn("978-0132350884");
        book.setPublicationYear(2008);
        entityManager.persistAndFlush(book);

        // When
        List<Book> foundBooks = bookRepository.findByAuthor("Robert Martin");

        // Then
        assertThat(foundBooks).hasSize(1);
        assertThat(foundBooks.get(0).getTitle()).isEqualTo("代码整洁之道");
    }
}
```

## 学习目标

- 使用`@DataJpaTest`的Spring Data JPA仓库测试
- 测试数据设置的TestEntityManager
- 自定义仓库查询方法
- 使用模拟仓库的服务层测试
- 使用`@SpringBootTest`的集成测试
- 数据库事务管理
- 异常处理和自定义异常
- 测试属性配置