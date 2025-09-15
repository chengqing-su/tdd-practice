# User Registration System

## Problem Description

Build a user registration system with validation, password encoding, and email uniqueness constraints. Learn Spring Security basics, custom validators, and integration testing with security components.

## Requirements

### User Entity
- User with username, email, password, firstName, lastName
- Email uniqueness constraint
- Password encryption using BCrypt
- User roles (USER, ADMIN)
- Account status (active, inactive, locked)

### Registration Features
- User registration endpoint
- Password strength validation
- Email format validation
- Duplicate email prevention
- Password encoding before storage

### Authentication Features
- Basic login endpoint
- JWT token generation (optional)
- User profile retrieval
- Password change functionality

### Validation Rules
- Email must be valid format and unique
- Password: minimum 8 characters, at least one uppercase, lowercase, digit
- Username: 3-20 characters, alphanumeric only
- First/Last name: required, 2-50 characters

## Project Setup

### Dependencies
```xml
<dependencies>
    <!-- Previous dependencies... -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-security</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.security</groupId>
        <artifactId>spring-security-test</artifactId>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>io.jsonwebtoken</groupId>
        <artifactId>jjwt-api</artifactId>
        <version>0.11.5</version>
    </dependency>
    <dependency>
        <groupId>io.jsonwebtoken</groupId>
        <artifactId>jjwt-impl</artifactId>
        <version>0.11.5</version>
        <scope>runtime</scope>
    </dependency>
</dependencies>
```

## User Entity Example

```java
@Entity
@Table(name = "users")
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(unique = true, nullable = false)
    @Size(min = 3, max = 20, message = "Username must be between 3 and 20 characters")
    @Pattern(regexp = "^[a-zA-Z0-9]+$", message = "Username must contain only letters and numbers")
    private String username;

    @Column(unique = true, nullable = false)
    @Email(message = "Email must be valid")
    private String email;

    @Column(nullable = false)
    @NotBlank(message = "Password is required")
    @ValidPassword
    private String password;

    @Column(nullable = false)
    @Size(min = 2, max = 50, message = "First name must be between 2 and 50 characters")
    private String firstName;

    @Column(nullable = false)
    @Size(min = 2, max = 50, message = "Last name must be between 2 and 50 characters")
    private String lastName;

    @Enumerated(EnumType.STRING)
    private Role role = Role.USER;

    @Enumerated(EnumType.STRING)
    private AccountStatus status = AccountStatus.ACTIVE;

    // constructors, getters, setters...
}
```

## Custom Password Validator

```java
@Target({ElementType.FIELD})
@Retention(RetentionPolicy.RUNTIME)
@Constraint(validatedBy = PasswordValidator.class)
public @interface ValidPassword {
    String message() default "Password must be at least 8 characters with uppercase, lowercase, and digit";
    Class<?>[] groups() default {};
    Class<? extends Payload>[] payload() default {};
}

public class PasswordValidator implements ConstraintValidator<ValidPassword, String> {
    private static final String PASSWORD_PATTERN =
        "^(?=.*[a-z])(?=.*[A-Z])(?=.*\\d)[a-zA-Z\\d@$!%*?&]{8,}$";

    private static final Pattern pattern = Pattern.compile(PASSWORD_PATTERN);

    @Override
    public boolean isValid(String password, ConstraintValidatorContext context) {
        return password != null && pattern.matcher(password).matches();
    }
}
```

## TDD Testing Approach

### 1. Repository Layer Tests
```java
@DataJpaTest
class UserRepositoryTest {

    @Autowired
    private TestEntityManager entityManager;

    @Autowired
    private UserRepository userRepository;

    @Test
    void shouldFindUserByEmail() {
        // Given
        User user = createUser();
        entityManager.persistAndFlush(user);

        // When
        Optional<User> found = userRepository.findByEmail("john@example.com");

        // Then
        assertThat(found).isPresent();
        assertThat(found.get().getUsername()).isEqualTo("johndoe");
    }

    @Test
    void shouldReturnEmptyWhenEmailNotExists() {
        // When
        Optional<User> found = userRepository.findByEmail("nonexistent@example.com");

        // Then
        assertThat(found).isEmpty();
    }

    @Test
    void shouldCheckIfEmailExists() {
        // Given
        User user = createUser();
        entityManager.persistAndFlush(user);

        // When
        boolean exists = userRepository.existsByEmail("john@example.com");

        // Then
        assertThat(exists).isTrue();
    }
}
```

### 2. Service Layer Tests
```java
@ExtendWith(MockitoExtension.class)
class UserServiceTest {

    @Mock
    private UserRepository userRepository;

    @Mock
    private PasswordEncoder passwordEncoder;

    @InjectMocks
    private UserService userService;

    @Test
    void shouldRegisterNewUser() {
        // Given
        RegisterRequest request = new RegisterRequest();
        request.setUsername("johndoe");
        request.setEmail("john@example.com");
        request.setPassword("Password123");
        request.setFirstName("John");
        request.setLastName("Doe");

        when(userRepository.existsByEmail("john@example.com")).thenReturn(false);
        when(userRepository.existsByUsername("johndoe")).thenReturn(false);
        when(passwordEncoder.encode("Password123")).thenReturn("$2a$10$encodedPassword");
        when(userRepository.save(any(User.class))).thenAnswer(invocation -> {
            User user = invocation.getArgument(0);
            user.setId(1L);
            return user;
        });

        // When
        User registeredUser = userService.registerUser(request);

        // Then
        assertThat(registeredUser.getId()).isEqualTo(1L);
        assertThat(registeredUser.getUsername()).isEqualTo("johndoe");
        assertThat(registeredUser.getEmail()).isEqualTo("john@example.com");
        assertThat(registeredUser.getPassword()).isEqualTo("$2a$10$encodedPassword");
        verify(passwordEncoder).encode("Password123");
    }

    @Test
    void shouldThrowExceptionWhenEmailAlreadyExists() {
        // Given
        RegisterRequest request = new RegisterRequest();
        request.setEmail("existing@example.com");

        when(userRepository.existsByEmail("existing@example.com")).thenReturn(true);

        // When & Then
        assertThatThrownBy(() -> userService.registerUser(request))
            .isInstanceOf(EmailAlreadyExistsException.class)
            .hasMessage("Email already exists: existing@example.com");
    }
}
```

### 3. Controller Tests with Security
```java
@WebMvcTest(UserController.class)
@Import(SecurityTestConfig.class)
class UserControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private UserService userService;

    @Test
    void shouldRegisterUser() throws Exception {
        // Given
        RegisterRequest request = new RegisterRequest();
        request.setUsername("johndoe");
        request.setEmail("john@example.com");
        request.setPassword("Password123");
        request.setFirstName("John");
        request.setLastName("Doe");

        User user = new User();
        user.setId(1L);
        user.setUsername("johndoe");
        user.setEmail("john@example.com");

        when(userService.registerUser(any(RegisterRequest.class))).thenReturn(user);

        // When & Then
        mockMvc.perform(post("/api/auth/register")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(request)))
                .andExpect(status().isCreated())
                .andExpect(jsonPath("$.id").value(1))
                .andExpected(jsonPath("$.username").value("johndoe"))
                .andExpect(jsonPath("$.email").value("john@example.com"));
    }

    @Test
    void shouldReturnBadRequestForInvalidPassword() throws Exception {
        // Given
        RegisterRequest request = new RegisterRequest();
        request.setUsername("johndoe");
        request.setEmail("john@example.com");
        request.setPassword("weak"); // Invalid password
        request.setFirstName("John");
        request.setLastName("Doe");

        // When & Then
        mockMvc.perform(post("/api/auth/register")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(request)))
                .andExpect(status().isBadRequest())
                .andExpect(jsonPath("$.errors").isArray())
                .andExpect(jsonPath("$.errors[0]").value(containsString("Password must be")));
    }

    @Test
    @WithMockUser(username = "johndoe", roles = "USER")
    void shouldGetUserProfile() throws Exception {
        // Given
        User user = createUser();
        when(userService.getUserByUsername("johndoe")).thenReturn(user);

        // When & Then
        mockMvc.perform(get("/api/users/profile"))
                .andExpect(status().isOk())
                .andExpected(jsonPath("$.username").value("johndoe"))
                .andExpect(jsonPath("$.email").value("john@example.com"));
    }

    @Test
    void shouldReturn401ForUnauthenticatedProfileAccess() throws Exception {
        // When & Then
        mockMvc.perform(get("/api/users/profile"))
                .andExpect(status().isUnauthorized());
    }
}
```

### 4. Integration Tests
```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@ActiveProfiles("test")
class UserRegistrationIntegrationTest {

    @Autowired
    private TestRestTemplate restTemplate;

    @Autowired
    private UserRepository userRepository;

    @BeforeEach
    void setUp() {
        userRepository.deleteAll();
    }

    @Test
    void shouldRegisterUserSuccessfully() {
        // Given
        RegisterRequest request = new RegisterRequest();
        request.setUsername("johndoe");
        request.setEmail("john@example.com");
        request.setPassword("Password123");
        request.setFirstName("John");
        request.setLastName("Doe");

        // When
        ResponseEntity<UserResponse> response = restTemplate.postForEntity(
            "/api/auth/register", request, UserResponse.class);

        // Then
        assertThat(response.getStatusCode()).isEqualTo(HttpStatus.CREATED);
        assertThat(response.getBody().getUsername()).isEqualTo("johndoe");

        // Verify in database
        Optional<User> savedUser = userRepository.findByEmail("john@example.com");
        assertThat(savedUser).isPresent();
        assertThat(savedUser.get().getPassword()).isNotEqualTo("Password123"); // Should be encoded
    }

    @Test
    void shouldPreventDuplicateEmailRegistration() {
        // Given - First user
        RegisterRequest request1 = createRegisterRequest("johndoe", "john@example.com");
        restTemplate.postForEntity("/api/auth/register", request1, UserResponse.class);

        // When - Second user with same email
        RegisterRequest request2 = createRegisterRequest("janedoe", "john@example.com");
        ResponseEntity<ErrorResponse> response = restTemplate.postForEntity(
            "/api/auth/register", request2, ErrorResponse.class);

        // Then
        assertThat(response.getStatusCode()).isEqualTo(HttpStatus.BAD_REQUEST);
        assertThat(response.getBody().getMessage()).contains("Email already exists");
    }
}
```

## Security Configuration

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .csrf(csrf -> csrf.disable())
            .sessionManagement(session -> session.sessionCreationPolicy(SessionCreationPolicy.STATELESS))
            .authorizeHttpRequests(authz -> authz
                .requestMatchers("/api/auth/**").permitAll()
                .requestMatchers("/h2-console/**").permitAll()
                .anyRequest().authenticated()
            );

        return http.build();
    }
}
```

## Learning Goals

- Spring Security integration and testing
- Password encoding with BCrypt
- Custom validation annotations
- Database constraints and unique indexes
- Security test configuration
- `@WithMockUser` for authenticated tests
- Exception handling for business rules
- Integration testing with security

## TDD Steps

1. **Red**: Write repository test for user lookup
2. **Green**: Create User entity and repository
3. **Red**: Write service test for registration
4. **Green**: Implement registration logic
5. **Red**: Write validation tests for password
6. **Green**: Create custom password validator
7. **Red**: Write controller test for registration endpoint
8. **Green**: Implement registration controller
9. **Red**: Write integration test for duplicate email
10. **Green**: Add uniqueness constraint handling
11. **Refactor**: Clean up and optimize

## Testing Checklist

### Validation Tests
- [ ] Valid user data accepted
- [ ] Invalid email format rejected
- [ ] Weak password rejected
- [ ] Short username rejected
- [ ] Missing required fields rejected

### Security Tests
- [ ] Password is encrypted before storage
- [ ] Duplicate email registration prevented
- [ ] Duplicate username registration prevented
- [ ] Profile access requires authentication
- [ ] Unauthorized requests return 401

### Integration Tests
- [ ] Full registration flow works end-to-end
- [ ] Database constraints are enforced
- [ ] Error responses have proper format
- [ ] Security configuration is applied correctly

---

## 问题描述

构建带验证、密码编码和邮箱唯一性约束的用户注册系统。学习Spring Security基础、自定义验证器和安全组件的集成测试。

## 需求

### User 实体
- 带用户名、邮箱、密码、姓、名的用户
- 邮箱唯一性约束
- 使用BCrypt的密码加密
- 用户角色（用户、管理员）
- 账户状态（活跃、非活跃、锁定）

### 注册功能
- 用户注册端点
- 密码强度验证
- 邮箱格式验证
- 重复邮箱防护
- 存储前密码编码

## 学习目标

- Spring Security集成和测试
- BCrypt密码编码
- 自定义验证注解
- 数据库约束和唯一索引
- 安全测试配置
- 认证测试的`@WithMockUser`
- 业务规则的异常处理
- 安全集成测试