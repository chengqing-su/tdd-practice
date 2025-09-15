# Todo API with Validation

## Problem Description

Create a comprehensive Todo API with advanced validation, date handling, priority management, and user association. This builds upon previous concepts while introducing more complex business rules and validation scenarios.

## Requirements

### Todo Entity
- Todo with title, description, dueDate, priority, status, assignedUser
- Custom validation for due dates (cannot be in the past)
- Priority levels: LOW, MEDIUM, HIGH, URGENT
- Status: TODO, IN_PROGRESS, DONE
- Association with User entity

### Advanced Validation
- Title: required, 5-100 characters
- Description: optional, max 500 characters
- Due date: cannot be in the past, required for HIGH/URGENT priority
- Priority-based validation rules
- Status transition validation (business rules)

### Business Rules
- Only assigned user can update their todos
- HIGH/URGENT priority todos must have due dates
- Cannot change status from DONE back to TODO directly
- Overdue todos should be flagged

### API Endpoints
- `GET /api/todos` - Get user's todos with filtering/sorting
- `GET /api/todos/{id}` - Get specific todo
- `POST /api/todos` - Create new todo
- `PUT /api/todos/{id}` - Update todo
- `DELETE /api/todos/{id}` - Delete todo
- `PATCH /api/todos/{id}/status` - Update todo status

## Project Setup

### Additional Dependencies
```xml
<dependencies>
    <!-- Previous dependencies... -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-validation</artifactId>
    </dependency>
    <dependency>
        <groupId>com.querydsl</groupId>
        <artifactId>querydsl-jpa</artifactId>
        <version>5.0.0</version>
    </dependency>
    <dependency>
        <groupId>com.querydsl</groupId>
        <artifactId>querydsl-apt</artifactId>
        <version>5.0.0</version>
        <scope>provided</scope>
    </dependency>
</dependencies>
```

## Todo Entity with Custom Validation

```java
@Entity
@Table(name = "todos")
@ValidTodo // Custom class-level validation
public class Todo {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @NotBlank(message = "Title is required")
    @Size(min = 5, max = 100, message = "Title must be between 5 and 100 characters")
    private String title;

    @Size(max = 500, message = "Description cannot exceed 500 characters")
    private String description;

    @FutureOrPresent(message = "Due date cannot be in the past")
    @ValidDueDate // Custom validation
    private LocalDateTime dueDate;

    @Enumerated(EnumType.STRING)
    @NotNull(message = "Priority is required")
    private Priority priority;

    @Enumerated(EnumType.STRING)
    @NotNull(message = "Status is required")
    private Status status = Status.TODO;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "user_id", nullable = false)
    private User assignedUser;

    @CreationTimestamp
    private LocalDateTime createdAt;

    @UpdateTimestamp
    private LocalDateTime updatedAt;

    // constructors, getters, setters...
}

public enum Priority {
    LOW, MEDIUM, HIGH, URGENT
}

public enum Status {
    TODO, IN_PROGRESS, DONE
}
```

## Custom Validators

### Due Date Validator
```java
@Target({ElementType.FIELD})
@Retention(RetentionPolicy.RUNTIME)
@Constraint(validatedBy = DueDateValidator.class)
public @interface ValidDueDate {
    String message() default "Due date validation failed";
    Class<?>[] groups() default {};
    Class<? extends Payload>[] payload() default {};
}

public class DueDateValidator implements ConstraintValidator<ValidDueDate, LocalDateTime> {
    @Override
    public boolean isValid(LocalDateTime dueDate, ConstraintValidatorContext context) {
        if (dueDate == null) {
            return true; // Let @NotNull handle null validation
        }
        return dueDate.isAfter(LocalDateTime.now());
    }
}
```

### Todo Class-Level Validator
```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Constraint(validatedBy = TodoValidator.class)
public @interface ValidTodo {
    String message() default "Invalid todo configuration";
    Class<?>[] groups() default {};
    Class<? extends Payload>[] payload() default {};
}

public class TodoValidator implements ConstraintValidator<ValidTodo, Todo> {
    @Override
    public boolean isValid(Todo todo, ConstraintValidatorContext context) {
        if (todo == null) {
            return true;
        }

        boolean isValid = true;

        // Rule: HIGH and URGENT priority todos must have due dates
        if ((todo.getPriority() == Priority.HIGH || todo.getPriority() == Priority.URGENT)
            && todo.getDueDate() == null) {
            context.buildConstraintViolationWithTemplate(
                "HIGH and URGENT priority todos must have a due date")
                .addPropertyNode("dueDate")
                .addConstraintViolation();
            isValid = false;
        }

        return isValid;
    }
}
```

## Service with Business Logic

```java
@Service
@Transactional
public class TodoService {

    private final TodoRepository todoRepository;
    private final UserService userService;

    public TodoService(TodoRepository todoRepository, UserService userService) {
        this.todoRepository = todoRepository;
        this.userService = userService;
    }

    public Todo createTodo(CreateTodoRequest request, String username) {
        User user = userService.getUserByUsername(username);

        Todo todo = new Todo();
        todo.setTitle(request.getTitle());
        todo.setDescription(request.getDescription());
        todo.setDueDate(request.getDueDate());
        todo.setPriority(request.getPriority());
        todo.setAssignedUser(user);

        return todoRepository.save(todo);
    }

    public Todo updateTodoStatus(Long todoId, Status newStatus, String username) {
        Todo todo = getTodoByIdAndUser(todoId, username);

        validateStatusTransition(todo.getStatus(), newStatus);

        todo.setStatus(newStatus);
        return todoRepository.save(todo);
    }

    private void validateStatusTransition(Status currentStatus, Status newStatus) {
        // Business rule: Cannot go from DONE back to TODO directly
        if (currentStatus == Status.DONE && newStatus == Status.TODO) {
            throw new InvalidStatusTransitionException(
                "Cannot change status from DONE to TODO directly. Set to IN_PROGRESS first."
            );
        }
    }

    public List<Todo> getTodos(String username, TodoFilter filter) {
        User user = userService.getUserByUsername(username);
        return todoRepository.findTodosWithFilter(user.getId(), filter);
    }

    private Todo getTodoByIdAndUser(Long todoId, String username) {
        User user = userService.getUserByUsername(username);
        return todoRepository.findByIdAndAssignedUser(todoId, user)
            .orElseThrow(() -> new TodoNotFoundException("Todo not found: " + todoId));
    }
}
```

## TDD Testing Examples

### 1. Validation Tests
```java
@ExtendWith(MockitoExtension.class)
class TodoValidationTest {

    private Validator validator;

    @BeforeEach
    void setUp() {
        ValidatorFactory factory = Validation.buildDefaultValidatorFactory();
        validator = factory.getValidator();
    }

    @Test
    void shouldRejectTodoWithShortTitle() {
        // Given
        Todo todo = new Todo();
        todo.setTitle("Hi"); // Too short
        todo.setPriority(Priority.LOW);

        // When
        Set<ConstraintViolation<Todo>> violations = validator.validate(todo);

        // Then
        assertThat(violations).hasSize(1);
        assertThat(violations.iterator().next().getMessage())
            .contains("Title must be between 5 and 100 characters");
    }

    @Test
    void shouldRejectHighPriorityTodoWithoutDueDate() {
        // Given
        Todo todo = new Todo();
        todo.setTitle("Important task");
        todo.setPriority(Priority.HIGH);
        todo.setDueDate(null); // Missing due date for high priority

        // When
        Set<ConstraintViolation<Todo>> violations = validator.validate(todo);

        // Then
        assertThat(violations).hasSize(1);
        assertThat(violations.iterator().next().getMessage())
            .contains("HIGH and URGENT priority todos must have a due date");
    }

    @Test
    void shouldRejectPastDueDate() {
        // Given
        Todo todo = new Todo();
        todo.setTitle("Past due task");
        todo.setPriority(Priority.LOW);
        todo.setDueDate(LocalDateTime.now().minusDays(1)); // Past date

        // When
        Set<ConstraintViolation<Todo>> violations = validator.validate(todo);

        // Then
        assertThat(violations).hasSize(1);
        assertThat(violations.iterator().next().getMessage())
            .contains("Due date cannot be in the past");
    }
}
```

### 2. Service Tests
```java
@ExtendWith(MockitoExtension.class)
class TodoServiceTest {

    @Mock
    private TodoRepository todoRepository;

    @Mock
    private UserService userService;

    @InjectMocks
    private TodoService todoService;

    @Test
    void shouldCreateTodoSuccessfully() {
        // Given
        User user = createUser();
        CreateTodoRequest request = new CreateTodoRequest();
        request.setTitle("Complete project");
        request.setDescription("Finish the TDD tutorial");
        request.setPriority(Priority.HIGH);
        request.setDueDate(LocalDateTime.now().plusDays(7));

        when(userService.getUserByUsername("johndoe")).thenReturn(user);
        when(todoRepository.save(any(Todo.class))).thenAnswer(invocation -> {
            Todo todo = invocation.getArgument(0);
            todo.setId(1L);
            return todo;
        });

        // When
        Todo result = todoService.createTodo(request, "johndoe");

        // Then
        assertThat(result.getId()).isEqualTo(1L);
        assertThat(result.getTitle()).isEqualTo("Complete project");
        assertThat(result.getAssignedUser()).isEqualTo(user);
        verify(todoRepository).save(any(Todo.class));
    }

    @Test
    void shouldThrowExceptionForInvalidStatusTransition() {
        // Given
        User user = createUser();
        Todo todo = createTodo();
        todo.setStatus(Status.DONE);
        todo.setAssignedUser(user);

        when(userService.getUserByUsername("johndoe")).thenReturn(user);
        when(todoRepository.findByIdAndAssignedUser(1L, user))
            .thenReturn(Optional.of(todo));

        // When & Then
        assertThatThrownBy(() -> todoService.updateTodoStatus(1L, Status.TODO, "johndoe"))
            .isInstanceOf(InvalidStatusTransitionException.class)
            .hasMessage("Cannot change status from DONE to TODO directly. Set to IN_PROGRESS first.");
    }
}
```

### 3. Controller Tests
```java
@WebMvcTest(TodoController.class)
@Import(SecurityTestConfig.class)
class TodoControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private TodoService todoService;

    @Test
    @WithMockUser(username = "johndoe")
    void shouldCreateTodo() throws Exception {
        // Given
        CreateTodoRequest request = new CreateTodoRequest();
        request.setTitle("Complete project");
        request.setDescription("Finish the TDD tutorial");
        request.setPriority(Priority.HIGH);
        request.setDueDate(LocalDateTime.now().plusDays(7));

        Todo todo = new Todo();
        todo.setId(1L);
        todo.setTitle(request.getTitle());
        todo.setPriority(request.getPriority());

        when(todoService.createTodo(any(CreateTodoRequest.class), eq("johndoe")))
            .thenReturn(todo);

        // When & Then
        mockMvc.perform(post("/api/todos")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(request)))
                .andExpect(status().isCreated())
                .andExpect(jsonPath("$.id").value(1))
                .andExpected(jsonPath("$.title").value("Complete project"))
                .andExpect(jsonPath("$.priority").value("HIGH"));
    }

    @Test
    @WithMockUser(username = "johndoe")
    void shouldReturnBadRequestForInvalidTodo() throws Exception {
        // Given
        CreateTodoRequest request = new CreateTodoRequest();
        request.setTitle("Hi"); // Too short
        request.setPriority(Priority.HIGH);
        // Missing due date for high priority

        // When & Then
        mockMvc.perform(post("/api/todos")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(request)))
                .andExpect(status().isBadRequest())
                .andExpected(jsonPath("$.errors").isArray())
                .andExpect(jsonPath("$.errors", hasSize(greaterThan(0))));
    }
}
```

## Repository with Custom Queries

```java
public interface TodoRepository extends JpaRepository<Todo, Long> {

    Optional<Todo> findByIdAndAssignedUser(Long id, User user);

    List<Todo> findByAssignedUserOrderByDueDateAsc(User user);

    @Query("SELECT t FROM Todo t WHERE t.assignedUser.id = :userId " +
           "AND (:status IS NULL OR t.status = :status) " +
           "AND (:priority IS NULL OR t.priority = :priority) " +
           "AND (:overdue IS NULL OR " +
           "     (:overdue = true AND t.dueDate < CURRENT_TIMESTAMP AND t.status != 'DONE') OR " +
           "     (:overdue = false AND (t.dueDate >= CURRENT_TIMESTAMP OR t.status = 'DONE')))")
    List<Todo> findTodosWithFilter(@Param("userId") Long userId,
                                   @Param("status") Status status,
                                   @Param("priority") Priority priority,
                                   @Param("overdue") Boolean overdue);
}
```

## Learning Goals

- Advanced Bean Validation with custom validators
- Class-level validation constraints
- Business rule validation in services
- Complex entity relationships and querying
- Date/time handling in validation and business logic
- Custom query methods with multiple parameters
- Status transition validation
- Security integration with user-specific data access

## TDD Steps

1. **Red**: Write validation tests for Todo entity
2. **Green**: Create Todo entity with validation annotations
3. **Red**: Write custom validator tests
4. **Green**: Implement custom validators
5. **Red**: Write service tests for business rules
6. **Green**: Implement service with business logic
7. **Red**: Write controller tests with validation scenarios
8. **Green**: Implement controller with proper error handling
9. **Red**: Write repository tests for custom queries
10. **Green**: Implement custom repository methods
11. **Red**: Write integration tests for complete flows
12. **Green**: Ensure all layers work together
13. **Refactor**: Optimize and clean up code

## Testing Checklist

### Validation Tests
- [ ] Required field validation works
- [ ] String length validation works
- [ ] Date validation prevents past dates
- [ ] Custom business rule validation works
- [ ] Priority-based validation rules work

### Service Tests
- [ ] Todo creation with user association
- [ ] Status transition validation
- [ ] User authorization for todo access
- [ ] Filtering and sorting functionality
- [ ] Business rule enforcement

### Integration Tests
- [ ] Full CRUD operations work end-to-end
- [ ] Validation errors return proper responses
- [ ] Security prevents unauthorized access
- [ ] Database queries perform correctly
- [ ] Complex filtering scenarios work

---

## 问题描述

创建带高级验证、日期处理、优先级管理和用户关联的综合Todo API。这建立在之前的概念基础上，同时引入更复杂的业务规则和验证场景。

## 学习目标

- 带自定义验证器的高级Bean验证
- 类级验证约束
- 服务中的业务规则验证
- 复杂实体关系和查询
- 验证和业务逻辑中的日期/时间处理
- 带多参数的自定义查询方法
- 状态转换验证
- 用户特定数据访问的安全集成