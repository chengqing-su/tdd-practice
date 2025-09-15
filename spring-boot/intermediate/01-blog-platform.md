# Blog Platform API

## Problem Description

Build a comprehensive blog platform with posts, comments, categories, and user roles. This exercise focuses on complex entity relationships, caching, pagination, search functionality, and role-based access control.

## Requirements

### Domain Model
- `Post` - Blog posts with title, content, author, category, published status
- `Comment` - Comments on posts with author and nested replies
- `Category` - Post categories with hierarchical structure
- `User` - Authors with roles (AUTHOR, EDITOR, ADMIN)
- `Tag` - Tags for posts (many-to-many relationship)

### Business Rules
- Only published posts are visible to public
- Authors can edit their own posts
- Editors can edit any post
- Comments can be nested (replies to comments)
- Posts must belong to a category
- Soft delete for posts and comments

### API Features
- CRUD operations for all entities
- Post search and filtering
- Pagination for all listings
- Comment threading
- Role-based permissions
- Caching for frequently accessed data

## Project Setup

### Dependencies
```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-cache</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-security</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-validation</artifactId>
    </dependency>
    <dependency>
        <groupId>com.github.ben-manes.caffeine</groupId>
        <artifactId>caffeine</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>org.testcontainers</groupId>
        <artifactId>postgresql</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>
```

## Domain Entities

### Post Entity
```java
@Entity
@Table(name = "posts")
@EntityListeners(AuditingEntityListener.class)
public class Post {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @NotBlank(message = "Title is required")
    @Size(max = 200, message = "Title cannot exceed 200 characters")
    @Column(nullable = false)
    private String title;

    @NotBlank(message = "Content is required")
    @Column(columnDefinition = "TEXT")
    private String content;

    @Column(columnDefinition = "TEXT")
    private String excerpt;

    @NotBlank(message = "Slug is required")
    @Size(max = 200)
    @Column(unique = true, nullable = false)
    private String slug;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "author_id", nullable = false)
    private User author;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "category_id", nullable = false)
    private Category category;

    @ManyToMany(fetch = FetchType.LAZY)
    @JoinTable(
        name = "post_tags",
        joinColumns = @JoinColumn(name = "post_id"),
        inverseJoinColumns = @JoinColumn(name = "tag_id")
    )
    private Set<Tag> tags = new HashSet<>();

    @OneToMany(mappedBy = "post", cascade = CascadeType.ALL, orphanRemoval = true)
    private List<Comment> comments = new ArrayList<>();

    private PostStatus status = PostStatus.DRAFT;

    @CreatedDate
    private LocalDateTime createdAt;

    @LastModifiedDate
    private LocalDateTime updatedAt;

    private LocalDateTime publishedAt;

    private boolean deleted = false;

    // constructors, getters, setters...
}

enum PostStatus {
    DRAFT, PUBLISHED, ARCHIVED
}
```

### Comment Entity
```java
@Entity
@Table(name = "comments")
public class Comment {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @NotBlank(message = "Comment content is required")
    @Size(max = 1000, message = "Comment cannot exceed 1000 characters")
    @Column(nullable = false)
    private String content;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "post_id", nullable = false)
    private Post post;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "author_id", nullable = false)
    private User author;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "parent_id")
    private Comment parent;

    @OneToMany(mappedBy = "parent", cascade = CascadeType.ALL, orphanRemoval = true)
    private List<Comment> replies = new ArrayList<>();

    @CreatedDate
    private LocalDateTime createdAt;

    private boolean deleted = false;

    // constructors, getters, setters...
}
```

## Service Layer with Caching

```java
@Service
@Transactional
public class PostService {

    private final PostRepository postRepository;
    private final CategoryService categoryService;
    private final TagService tagService;

    public PostService(PostRepository postRepository, CategoryService categoryService, TagService tagService) {
        this.postRepository = postRepository;
        this.categoryService = categoryService;
        this.tagService = tagService;
    }

    @Cacheable(value = "posts", key = "#slug")
    @Transactional(readOnly = true)
    public Post getPublishedPostBySlug(String slug) {
        return postRepository.findBySlugAndStatusAndDeletedFalse(slug, PostStatus.PUBLISHED)
            .orElseThrow(() -> new PostNotFoundException("Post not found: " + slug));
    }

    @Cacheable(value = "posts-page", key = "#pageable.pageNumber + '_' + #pageable.pageSize + '_' + #categoryId")
    @Transactional(readOnly = true)
    public Page<Post> getPublishedPosts(Pageable pageable, Long categoryId) {
        if (categoryId != null) {
            return postRepository.findByStatusAndCategoryIdAndDeletedFalse(
                PostStatus.PUBLISHED, categoryId, pageable);
        }
        return postRepository.findByStatusAndDeletedFalse(PostStatus.PUBLISHED, pageable);
    }

    @CacheEvict(value = {"posts", "posts-page"}, allEntries = true)
    public Post createPost(CreatePostRequest request, User author) {
        validateCreatePostRequest(request);

        Post post = new Post();
        post.setTitle(request.getTitle());
        post.setContent(request.getContent());
        post.setExcerpt(generateExcerpt(request.getContent()));
        post.setSlug(generateSlug(request.getTitle()));
        post.setAuthor(author);
        post.setCategory(categoryService.getCategoryById(request.getCategoryId()));
        post.setStatus(request.getStatus());

        if (request.getTagNames() != null && !request.getTagNames().isEmpty()) {
            Set<Tag> tags = tagService.findOrCreateTags(request.getTagNames());
            post.setTags(tags);
        }

        if (request.getStatus() == PostStatus.PUBLISHED) {
            post.setPublishedAt(LocalDateTime.now());
        }

        return postRepository.save(post);
    }

    @CacheEvict(value = {"posts", "posts-page"}, allEntries = true)
    public Post updatePost(Long postId, UpdatePostRequest request, User currentUser) {
        Post post = postRepository.findByIdAndDeletedFalse(postId)
            .orElseThrow(() -> new PostNotFoundException("Post not found: " + postId));

        validateUpdatePermission(post, currentUser);

        post.setTitle(request.getTitle());
        post.setContent(request.getContent());
        post.setExcerpt(generateExcerpt(request.getContent()));

        if (request.getCategoryId() != null) {
            post.setCategory(categoryService.getCategoryById(request.getCategoryId()));
        }

        // Handle status change
        if (request.getStatus() != null && !request.getStatus().equals(post.getStatus())) {
            updatePostStatus(post, request.getStatus());
        }

        return postRepository.save(post);
    }

    private void validateUpdatePermission(Post post, User user) {
        if (user.getRole() == Role.ADMIN || user.getRole() == Role.EDITOR) {
            return; // Admins and editors can edit any post
        }

        if (user.getRole() == Role.AUTHOR && post.getAuthor().getId().equals(user.getId())) {
            return; // Authors can edit their own posts
        }

        throw new AccessDeniedException("You don't have permission to edit this post");
    }

    @Transactional(readOnly = true)
    public Page<Post> searchPosts(String query, Pageable pageable) {
        return postRepository.findByTitleContainingIgnoreCaseOrContentContainingIgnoreCaseAndStatusAndDeletedFalse(
            query, query, PostStatus.PUBLISHED, pageable);
    }

    private String generateSlug(String title) {
        String baseSlug = title.toLowerCase()
            .replaceAll("[^a-z0-9\\s]", "")
            .replaceAll("\\s+", "-");

        String slug = baseSlug;
        int counter = 1;
        while (postRepository.existsBySlug(slug)) {
            slug = baseSlug + "-" + counter++;
        }
        return slug;
    }

    private String generateExcerpt(String content) {
        if (content.length() <= 150) {
            return content;
        }
        return content.substring(0, 147) + "...";
    }
}
```

## TDD Testing Examples

### 1. Repository Layer Tests
```java
@DataJpaTest
@TestPropertySource(properties = {
    "spring.jpa.hibernate.ddl-auto=create-drop",
    "spring.test.database.replace=none"
})
class PostRepositoryTest {

    @Autowired
    private TestEntityManager entityManager;

    @Autowired
    private PostRepository postRepository;

    @Test
    void shouldFindPublishedPostBySlug() {
        // Given
        User author = createAndSaveUser("author@example.com", Role.AUTHOR);
        Category category = createAndSaveCategory("Technology");
        Post post = createPost("Test Post", "test-post", PostStatus.PUBLISHED, author, category);
        entityManager.persistAndFlush(post);

        // When
        Optional<Post> result = postRepository.findBySlugAndStatusAndDeletedFalse("test-post", PostStatus.PUBLISHED);

        // Then
        assertThat(result).isPresent();
        assertThat(result.get().getTitle()).isEqualTo("Test Post");
        assertThat(result.get().getStatus()).isEqualTo(PostStatus.PUBLISHED);
    }

    @Test
    void shouldNotFindDraftPostBySlugWhenSearchingForPublished() {
        // Given
        User author = createAndSaveUser("author@example.com", Role.AUTHOR);
        Category category = createAndSaveCategory("Technology");
        Post post = createPost("Draft Post", "draft-post", PostStatus.DRAFT, author, category);
        entityManager.persistAndFlush(post);

        // When
        Optional<Post> result = postRepository.findBySlugAndStatusAndDeletedFalse("draft-post", PostStatus.PUBLISHED);

        // Then
        assertThat(result).isEmpty();
    }

    @Test
    void shouldFindPostsByCategory() {
        // Given
        User author = createAndSaveUser("author@example.com", Role.AUTHOR);
        Category techCategory = createAndSaveCategory("Technology");
        Category businessCategory = createAndSaveCategory("Business");

        Post techPost = createPost("Tech Post", "tech-post", PostStatus.PUBLISHED, author, techCategory);
        Post businessPost = createPost("Business Post", "business-post", PostStatus.PUBLISHED, author, businessCategory);

        entityManager.persistAndFlush(techPost);
        entityManager.persistAndFlush(businessPost);

        Pageable pageable = PageRequest.of(0, 10);

        // When
        Page<Post> result = postRepository.findByStatusAndCategoryIdAndDeletedFalse(
            PostStatus.PUBLISHED, techCategory.getId(), pageable);

        // Then
        assertThat(result.getContent()).hasSize(1);
        assertThat(result.getContent().get(0).getCategory().getName()).isEqualTo("Technology");
    }

    @Test
    void shouldSearchPostsByTitleAndContent() {
        // Given
        User author = createAndSaveUser("author@example.com", Role.AUTHOR);
        Category category = createAndSaveCategory("Technology");

        Post post1 = createPost("Spring Boot Tutorial", "spring-tutorial", PostStatus.PUBLISHED, author, category);
        post1.setContent("Learn Spring Boot framework");

        Post post2 = createPost("Java Basics", "java-basics", PostStatus.PUBLISHED, author, category);
        post2.setContent("Introduction to Java programming");

        entityManager.persistAndFlush(post1);
        entityManager.persistAndFlush(post2);

        Pageable pageable = PageRequest.of(0, 10);

        // When
        Page<Post> result = postRepository.findByTitleContainingIgnoreCaseOrContentContainingIgnoreCaseAndStatusAndDeletedFalse(
            "spring", "spring", PostStatus.PUBLISHED, pageable);

        // Then
        assertThat(result.getContent()).hasSize(1);
        assertThat(result.getContent().get(0).getTitle()).isEqualTo("Spring Boot Tutorial");
    }

    private User createAndSaveUser(String email, Role role) {
        User user = new User();
        user.setUsername(email.split("@")[0]);
        user.setEmail(email);
        user.setPassword("encodedPassword");
        user.setRole(role);
        return entityManager.persistAndFlush(user);
    }

    private Category createAndSaveCategory(String name) {
        Category category = new Category();
        category.setName(name);
        category.setSlug(name.toLowerCase().replace(" ", "-"));
        return entityManager.persistAndFlush(category);
    }

    private Post createPost(String title, String slug, PostStatus status, User author, Category category) {
        Post post = new Post();
        post.setTitle(title);
        post.setSlug(slug);
        post.setContent("Sample content");
        post.setExcerpt("Sample excerpt");
        post.setStatus(status);
        post.setAuthor(author);
        post.setCategory(category);
        if (status == PostStatus.PUBLISHED) {
            post.setPublishedAt(LocalDateTime.now());
        }
        return post;
    }
}
```

### 2. Service Layer Tests with Caching
```java
@ExtendWith(MockitoExtension.class)
class PostServiceTest {

    @Mock
    private PostRepository postRepository;

    @Mock
    private CategoryService categoryService;

    @Mock
    private TagService tagService;

    @InjectMocks
    private PostService postService;

    @Test
    void shouldCreatePostSuccessfully() {
        // Given
        User author = createUser(Role.AUTHOR);
        Category category = createCategory();
        Set<Tag> tags = Set.of(createTag("spring"), createTag("java"));

        CreatePostRequest request = new CreatePostRequest();
        request.setTitle("Test Post");
        request.setContent("This is a test post content");
        request.setCategoryId(1L);
        request.setTagNames(Set.of("spring", "java"));
        request.setStatus(PostStatus.PUBLISHED);

        when(categoryService.getCategoryById(1L)).thenReturn(category);
        when(tagService.findOrCreateTags(Set.of("spring", "java"))).thenReturn(tags);
        when(postRepository.save(any(Post.class))).thenAnswer(invocation -> {
            Post post = invocation.getArgument(0);
            post.setId(1L);
            return post;
        });

        // When
        Post result = postService.createPost(request, author);

        // Then
        assertThat(result.getId()).isEqualTo(1L);
        assertThat(result.getTitle()).isEqualTo("Test Post");
        assertThat(result.getSlug()).isEqualTo("test-post");
        assertThat(result.getStatus()).isEqualTo(PostStatus.PUBLISHED);
        assertThat(result.getPublishedAt()).isNotNull();
        verify(postRepository).save(any(Post.class));
    }

    @Test
    void shouldAllowAuthorToEditTheirOwnPost() {
        // Given
        User author = createUser(Role.AUTHOR);
        author.setId(1L);

        Post post = createPost();
        post.setAuthor(author);

        UpdatePostRequest request = new UpdatePostRequest();
        request.setTitle("Updated Title");
        request.setContent("Updated content");

        when(postRepository.findByIdAndDeletedFalse(1L)).thenReturn(Optional.of(post));
        when(postRepository.save(any(Post.class))).thenAnswer(invocation -> invocation.getArgument(0));

        // When
        Post result = postService.updatePost(1L, request, author);

        // Then
        assertThat(result.getTitle()).isEqualTo("Updated Title");
        assertThat(result.getContent()).isEqualTo("Updated content");
        verify(postRepository).save(post);
    }

    @Test
    void shouldThrowExceptionWhenAuthorTriesToEditOthersPost() {
        // Given
        User author = createUser(Role.AUTHOR);
        author.setId(1L);

        User anotherAuthor = createUser(Role.AUTHOR);
        anotherAuthor.setId(2L);

        Post post = createPost();
        post.setAuthor(anotherAuthor);

        UpdatePostRequest request = new UpdatePostRequest();
        request.setTitle("Updated Title");

        when(postRepository.findByIdAndDeletedFalse(1L)).thenReturn(Optional.of(post));

        // When & Then
        assertThatThrownBy(() -> postService.updatePost(1L, request, author))
            .isInstanceOf(AccessDeniedException.class)
            .hasMessage("You don't have permission to edit this post");
    }

    @Test
    void shouldAllowEditorToEditAnyPost() {
        // Given
        User editor = createUser(Role.EDITOR);
        editor.setId(1L);

        User author = createUser(Role.AUTHOR);
        author.setId(2L);

        Post post = createPost();
        post.setAuthor(author);

        UpdatePostRequest request = new UpdatePostRequest();
        request.setTitle("Editor Updated Title");

        when(postRepository.findByIdAndDeletedFalse(1L)).thenReturn(Optional.of(post));
        when(postRepository.save(any(Post.class))).thenAnswer(invocation -> invocation.getArgument(0));

        // When
        Post result = postService.updatePost(1L, request, editor);

        // Then
        assertThat(result.getTitle()).isEqualTo("Editor Updated Title");
        verify(postRepository).save(post);
    }

    private User createUser(Role role) {
        User user = new User();
        user.setRole(role);
        user.setUsername("testuser");
        user.setEmail("test@example.com");
        return user;
    }

    private Category createCategory() {
        Category category = new Category();
        category.setId(1L);
        category.setName("Technology");
        return category;
    }

    private Tag createTag(String name) {
        Tag tag = new Tag();
        tag.setName(name);
        return tag;
    }

    private Post createPost() {
        Post post = new Post();
        post.setId(1L);
        post.setTitle("Original Title");
        post.setContent("Original content");
        return post;
    }
}
```

### 3. Integration Tests with Caching
```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@ActiveProfiles("test")
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

    @Autowired
    private TestRestTemplate restTemplate;

    @Autowired
    private PostRepository postRepository;

    @Autowired
    private UserRepository userRepository;

    @Autowired
    private CategoryRepository categoryRepository;

    @Autowired
    private CacheManager cacheManager;

    @BeforeEach
    void setUp() {
        postRepository.deleteAll();
        userRepository.deleteAll();
        categoryRepository.deleteAll();
        clearCaches();
    }

    @Test
    void shouldCreateAndRetrievePost() {
        // Given
        User author = createAndSaveUser("author@example.com", Role.AUTHOR);
        Category category = createAndSaveCategory("Technology");

        CreatePostRequest request = new CreatePostRequest();
        request.setTitle("Integration Test Post");
        request.setContent("This is an integration test");
        request.setCategoryId(category.getId());
        request.setStatus(PostStatus.PUBLISHED);

        String jwt = generateJwtToken(author);

        // When
        HttpHeaders headers = new HttpHeaders();
        headers.setBearerAuth(jwt);
        HttpEntity<CreatePostRequest> entity = new HttpEntity<>(request, headers);

        ResponseEntity<PostResponse> createResponse = restTemplate.postForEntity(
            "/api/posts", entity, PostResponse.class);

        // Then
        assertThat(createResponse.getStatusCode()).isEqualTo(HttpStatus.CREATED);
        assertThat(createResponse.getBody().getTitle()).isEqualTo("Integration Test Post");

        // Verify post can be retrieved
        ResponseEntity<PostResponse> getResponse = restTemplate.getForEntity(
            "/api/posts/" + createResponse.getBody().getSlug(), PostResponse.class);

        assertThat(getResponse.getStatusCode()).isEqualTo(HttpStatus.OK);
        assertThat(getResponse.getBody().getTitle()).isEqualTo("Integration Test Post");
    }

    @Test
    void shouldCachePostRetrieval() {
        // Given
        User author = createAndSaveUser("author@example.com", Role.AUTHOR);
        Category category = createAndSaveCategory("Technology");
        Post post = createAndSavePost("Cached Post", "cached-post", author, category);

        // When - First request (should cache)
        ResponseEntity<PostResponse> response1 = restTemplate.getForEntity(
            "/api/posts/" + post.getSlug(), PostResponse.class);

        // Then - Verify response and cache
        assertThat(response1.getStatusCode()).isEqualTo(HttpStatus.OK);
        Cache postsCache = cacheManager.getCache("posts");
        assertThat(postsCache.get(post.getSlug())).isNotNull();

        // When - Second request (should use cache)
        ResponseEntity<PostResponse> response2 = restTemplate.getForEntity(
            "/api/posts/" + post.getSlug(), PostResponse.class);

        // Then
        assertThat(response2.getStatusCode()).isEqualTo(HttpStatus.OK);
        assertThat(response2.getBody().getTitle()).isEqualTo("Cached Post");
    }

    @Test
    void shouldEnforceRoleBasedAccessControl() {
        // Given
        User author1 = createAndSaveUser("author1@example.com", Role.AUTHOR);
        User author2 = createAndSaveUser("author2@example.com", Role.AUTHOR);
        Category category = createAndSaveCategory("Technology");
        Post post = createAndSavePost("Author1 Post", "author1-post", author1, category);

        UpdatePostRequest request = new UpdatePostRequest();
        request.setTitle("Unauthorized Update");

        String jwt = generateJwtToken(author2);
        HttpHeaders headers = new HttpHeaders();
        headers.setBearerAuth(jwt);
        HttpEntity<UpdatePostRequest> entity = new HttpEntity<>(request, headers);

        // When
        ResponseEntity<String> response = restTemplate.exchange(
            "/api/posts/" + post.getId(), HttpMethod.PUT, entity, String.class);

        // Then
        assertThat(response.getStatusCode()).isEqualTo(HttpStatus.FORBIDDEN);
    }

    private void clearCaches() {
        cacheManager.getCacheNames().forEach(cacheName -> {
            Cache cache = cacheManager.getCache(cacheName);
            if (cache != null) {
                cache.clear();
            }
        });
    }

    // Helper methods for creating test data...
}
```

## Learning Goals

- Complex entity relationships (OneToMany, ManyToMany)
- Caching strategies with Spring Cache
- Role-based access control implementation
- Soft delete patterns
- Full-text search implementation
- Pagination and sorting
- Integration testing with TestContainers
- Performance testing and optimization

## TDD Steps

1. **Red**: Write repository tests for complex queries
2. **Green**: Implement repository with custom query methods
3. **Red**: Write service tests for business logic
4. **Green**: Implement service with caching
5. **Red**: Write controller tests with security
6. **Green**: Implement REST controllers
7. **Red**: Write integration tests for full workflows
8. **Green**: Ensure all layers work together
9. **Refactor**: Optimize queries and caching

## Advanced Testing Topics

- Cache testing strategies
- Security integration testing
- Performance testing with large datasets
- Transactional testing patterns
- Custom validation testing

---

## 问题描述

构建带帖子、评论、分类和用户角色的综合博客平台。此练习专注于复杂实体关系、缓存、分页、搜索功能和基于角色的访问控制。

## 学习目标

- 复杂实体关系（OneToMany, ManyToMany）
- Spring Cache缓存策略
- 基于角色的访问控制实现
- 软删除模式
- 全文搜索实现
- 分页和排序
- TestContainers集成测试
- 性能测试和优化