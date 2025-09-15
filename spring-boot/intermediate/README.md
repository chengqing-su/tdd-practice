# Spring Boot Intermediate Problems

These intermediate problems focus on complex business logic, advanced Spring Boot features, and real-world application patterns. Perfect for developers ready to tackle enterprise-level challenges.

[中文版本](#spring-boot-中级问题)

## Problems

1. **[Blog Platform API](01-blog-platform.md)** - Complex entity relationships, caching, role-based access control
2. **[Inventory Management System](02-inventory-management-system.md)** - Transaction management, event-driven architecture, business rules

## What You'll Learn

### Advanced Spring Boot Features
- **Spring Cache** - Caching strategies and cache eviction
- **Spring Events** - Event-driven architecture patterns
- **Spring Transaction Management** - Complex transactional scenarios
- **Spring Security** - Advanced security configurations and role-based access
- **Spring Data JPA** - Complex queries, specifications, and custom repositories

### Enterprise Patterns
- **Domain-Driven Design** - Rich domain models and business logic
- **Event-Driven Architecture** - Asynchronous processing and event handling
- **CQRS** - Command Query Responsibility Segregation
- **Soft Delete** - Data retention and recovery patterns
- **Audit Trail** - Complete change tracking

### Advanced Testing Techniques
- **TestContainers** - Integration testing with real databases
- **Performance Testing** - Load testing and optimization
- **Security Testing** - Authentication and authorization testing
- **Event Testing** - Asynchronous event verification
- **Cache Testing** - Cache behavior verification

### Database Patterns
- **Complex Relationships** - OneToMany, ManyToMany, hierarchical data
- **Custom Query Methods** - Advanced JPQL and native queries
- **Pagination and Sorting** - Large dataset handling
- **Database Constraints** - Data integrity enforcement
- **Optimistic Locking** - Concurrent access control

## Prerequisites

- Completion of [Beginner Problems](../beginner/)
- Understanding of Spring Boot fundamentals
- JPA and Hibernate knowledge
- Basic understanding of distributed systems concepts
- Familiarity with design patterns

## Key Learning Objectives

### Business Logic Complexity
These problems introduce complex business rules that mirror real-world applications:
- Multi-step workflows with validation
- State transitions and business constraints
- Cross-entity business rules
- Complex calculations and aggregations

### Data Consistency
Learn to maintain data integrity across complex operations:
- Transactional boundaries
- Rollback scenarios
- Concurrent access patterns
- Event consistency

### Performance Optimization
Understand how to build performant applications:
- Caching strategies
- Query optimization
- Lazy loading patterns
- Pagination implementation

### Testing Strategies
Master advanced testing techniques:
- Integration testing with external dependencies
- Performance testing approaches
- Security testing patterns
- Event-driven system testing

## Problem Complexity

### Blog Platform API
**Complexity**: ⭐⭐⭐⭐
**Focus Areas**:
- Complex entity relationships (posts, comments, users, categories)
- Hierarchical comment threading
- Role-based permissions (author, editor, admin)
- Caching with Spring Cache
- Full-text search capabilities
- Soft delete implementation

**Key Challenges**:
- Managing circular references in JPA entities
- Implementing efficient caching strategies
- Testing role-based security
- Handling hierarchical data structures

### Inventory Management System
**Complexity**: ⭐⭐⭐⭐⭐
**Focus Areas**:
- Complex business rules (stock levels, reservations)
- Event-driven architecture with Spring Events
- Multi-step transactions across entities
- Concurrent access to shared resources
- Audit trail implementation
- Business constraint enforcement

**Key Challenges**:
- Preventing negative stock levels
- Handling concurrent inventory updates
- Implementing complex business workflows
- Testing transactional consistency

## Advanced Testing Patterns

### Integration Testing with TestContainers
```java
@SpringBootTest
@Testcontainers
class BlogPlatformIntegrationTest {

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
    void shouldPersistDataCorrectly() {
        // Test with real PostgreSQL database
    }
}
```

### Event-Driven Testing
```java
@SpringBootTest
class EventDrivenTest {

    @MockBean
    private ApplicationEventPublisher eventPublisher;

    @Test
    void shouldPublishEventOnStockUpdate() {
        // Given
        StockUpdateRequest request = new StockUpdateRequest();

        // When
        inventoryService.updateStock(request);

        // Then
        verify(eventPublisher).publishEvent(any(StockUpdatedEvent.class));
    }

    @EventListener
    @Async
    public void handleStockUpdate(StockUpdatedEvent event) {
        // Event handling logic
    }
}
```

### Cache Testing
```java
@SpringBootTest
class CacheTest {

    @Autowired
    private PostService postService;

    @Autowired
    private CacheManager cacheManager;

    @Test
    void shouldCachePostRetrieval() {
        // Given
        String slug = "test-post";

        // When - First call
        Post post1 = postService.getPostBySlug(slug);

        // Then - Verify cached
        Cache cache = cacheManager.getCache("posts");
        assertThat(cache.get(slug)).isNotNull();

        // When - Second call
        Post post2 = postService.getPostBySlug(slug);

        // Then - Should return cached instance
        assertThat(post1).isSameAs(post2);
    }
}
```

## Performance Considerations

### N+1 Query Problem Prevention
```java
// Use JOIN FETCH to avoid N+1 queries
@Query("SELECT p FROM Post p JOIN FETCH p.comments WHERE p.id = :id")
Optional<Post> findByIdWithComments(@Param("id") Long id);

// Use @EntityGraph for complex fetch strategies
@EntityGraph(attributePaths = {"author", "category", "tags"})
List<Post> findAll();
```

### Pagination Best Practices
```java
@RestController
public class PostController {

    @GetMapping("/api/posts")
    public Page<PostDTO> getPosts(
            @PageableDefault(size = 20, sort = "createdAt", direction = Sort.Direction.DESC)
            Pageable pageable) {
        return postService.getAllPosts(pageable);
    }
}
```

### Caching Strategies
```java
@Service
public class PostService {

    @Cacheable(value = "posts", key = "#slug")
    public Post getPostBySlug(String slug) {
        return postRepository.findBySlug(slug);
    }

    @CacheEvict(value = "posts", allEntries = true)
    public Post updatePost(Post post) {
        return postRepository.save(post);
    }
}
```

## Security Patterns

### Method-Level Security
```java
@Service
@PreAuthorize("hasRole('USER')")
public class PostService {

    @PreAuthorize("hasRole('ADMIN') or @postService.isOwner(#postId, authentication.name)")
    public Post updatePost(Long postId, UpdatePostRequest request) {
        // Implementation
    }

    public boolean isOwner(Long postId, String username) {
        return postRepository.findById(postId)
            .map(post -> post.getAuthor().getUsername().equals(username))
            .orElse(false);
    }
}
```

### Custom Security Expressions
```java
@Component("postSecurity")
public class PostSecurityService {

    public boolean canEdit(Long postId, Authentication auth) {
        // Custom authorization logic
        return hasRole(auth, "EDITOR") || isAuthor(postId, auth.getName());
    }
}

// Usage in controller
@PreAuthorize("@postSecurity.canEdit(#postId, authentication)")
@PutMapping("/posts/{postId}")
public ResponseEntity<PostDTO> updatePost(@PathVariable Long postId, @RequestBody UpdatePostRequest request) {
    // Implementation
}
```

## Best Practices

### Error Handling
- Implement global exception handling with `@ControllerAdvice`
- Create custom business exceptions
- Provide meaningful error messages
- Log errors appropriately

### Data Validation
- Use Bean Validation annotations
- Create custom validators for complex rules
- Validate at multiple layers (controller, service)
- Handle validation errors gracefully

### Transaction Management
- Use `@Transactional` appropriately
- Understand transaction boundaries
- Handle rollback scenarios
- Test transactional behavior

### Testing Strategy
- Use appropriate test slices
- Mock external dependencies
- Test business logic thoroughly
- Include integration tests for critical paths

## Next Steps

After completing intermediate problems:
1. Move to [Advanced Problems](../advanced/) for distributed systems
2. Study microservices architecture patterns
3. Learn about reactive programming with WebFlux
4. Explore cloud-native development patterns
5. Master observability and monitoring

---

# Spring Boot 中级问题

这些中级问题专注于复杂业务逻辑、高级Spring Boot功能和真实世界应用模式。非常适合准备应对企业级挑战的开发者。

## 问题列表

1. **[博客平台API](01-blog-platform.md)** - 复杂实体关系、缓存、基于角色的访问控制
2. **[库存管理系统](02-inventory-management-system.md)** - 事务管理、事件驱动架构、业务规则

## 你将学到什么

### 高级Spring Boot功能
- **Spring Cache** - 缓存策略和缓存驱逐
- **Spring Events** - 事件驱动架构模式
- **Spring Transaction Management** - 复杂事务场景
- **Spring Security** - 高级安全配置和基于角色的访问
- **Spring Data JPA** - 复杂查询、规范和自定义仓库

### 企业模式
- **领域驱动设计** - 丰富的领域模型和业务逻辑
- **事件驱动架构** - 异步处理和事件处理
- **CQRS** - 命令查询职责分离
- **软删除** - 数据保留和恢复模式
- **审计跟踪** - 完整变更跟踪

### 高级测试技术
- **TestContainers** - 真实数据库集成测试
- **性能测试** - 负载测试和优化
- **安全测试** - 身份验证和授权测试
- **事件测试** - 异步事件验证
- **缓存测试** - 缓存行为验证

### 数据库模式
- **复杂关系** - OneToMany, ManyToMany, 层次数据
- **自定义查询方法** - 高级JPQL和原生查询
- **分页和排序** - 大数据集处理
- **数据库约束** - 数据完整性强制
- **乐观锁** - 并发访问控制

## 前置条件

- 完成[初级问题](../beginner/)
- 理解Spring Boot基础
- JPA和Hibernate知识
- 分布式系统概念基本理解
- 熟悉设计模式

## 最佳实践

### 错误处理
- 使用`@ControllerAdvice`实现全局异常处理
- 创建自定义业务异常
- 提供有意义的错误消息
- 适当记录错误日志

### 数据验证
- 使用Bean Validation注解
- 为复杂规则创建自定义验证器
- 在多层验证（控制器、服务）
- 优雅处理验证错误

### 事务管理
- 适当使用`@Transactional`
- 理解事务边界
- 处理回滚场景
- 测试事务行为

### 测试策略
- 使用适当的测试切片
- 模拟外部依赖
- 彻底测试业务逻辑
- 包含关键路径的集成测试

## 下一步

完成中级问题后：
1. 转向[高级问题](../advanced/)学习分布式系统
2. 研究微服务架构模式
3. 学习WebFlux响应式编程
4. 探索云原生开发模式
5. 掌握可观测性和监控