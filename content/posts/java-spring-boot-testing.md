---
title: "Testing trong Spring Boot - Unit Test và Integration Test"
date: 2024-01-04T08:30:00+07:00
draft: false
tags: ['java', 'spring-boot', 'testing', 'junit', 'mockito']
description: "Hướng dẫn chi tiết về testing trong Spring Boot từ unit test đến integration test"
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: true
ShowCodeCopyButtons: true
toc: true
---

# Testing trong Spring Boot - Unit Test và Integration Test

Testing là một phần quan trọng trong phát triển phần mềm. Spring Boot cung cấp các công cụ mạnh mẽ để viết unit test và integration test. Trong bài viết này, tôi sẽ hướng dẫn bạn cách testing hiệu quả.

## Tại Sao Cần Testing?

- **Quality Assurance**: Đảm bảo chất lượng code
- **Regression Prevention**: Ngăn chặn lỗi khi thay đổi code
- **Documentation**: Test cases là tài liệu sống
- **Confidence**: Tự tin khi refactor code
- **CI/CD**: Tích hợp vào quy trình deployment

## Dependencies cho Testing

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>org.testcontainers</groupId>
        <artifactId>junit-jupiter</artifactId>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>org.testcontainers</groupId>
        <artifactId>mysql</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>
```

## Unit Testing

### 1. Service Layer Testing

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
    void shouldCreateUserSuccessfully() {
        // Given
        CreateUserRequest request = new CreateUserRequest();
        request.setUsername("johndoe");
        request.setEmail("john@example.com");
        request.setPassword("password123");
        request.setFirstName("John");
        request.setLastName("Doe");
        
        User savedUser = new User();
        savedUser.setId(1L);
        savedUser.setUsername("johndoe");
        savedUser.setEmail("john@example.com");
        
        when(userRepository.existsByUsername("johndoe")).thenReturn(false);
        when(userRepository.existsByEmail("john@example.com")).thenReturn(false);
        when(passwordEncoder.encode("password123")).thenReturn("encodedPassword");
        when(userRepository.save(any(User.class))).thenReturn(savedUser);
        
        // When
        User result = userService.createUser(request);
        
        // Then
        assertThat(result).isNotNull();
        assertThat(result.getUsername()).isEqualTo("johndoe");
        assertThat(result.getEmail()).isEqualTo("john@example.com");
        
        verify(userRepository).existsByUsername("johndoe");
        verify(userRepository).existsByEmail("john@example.com");
        verify(passwordEncoder).encode("password123");
        verify(userRepository).save(any(User.class));
    }
    
    @Test
    void shouldThrowExceptionWhenUsernameExists() {
        // Given
        CreateUserRequest request = new CreateUserRequest();
        request.setUsername("johndoe");
        request.setEmail("john@example.com");
        request.setPassword("password123");
        request.setFirstName("John");
        request.setLastName("Doe");
        
        when(userRepository.existsByUsername("johndoe")).thenReturn(true);
        
        // When & Then
        assertThatThrownBy(() -> userService.createUser(request))
            .isInstanceOf(RuntimeException.class)
            .hasMessage("Username already exists");
        
        verify(userRepository).existsByUsername("johndoe");
        verify(userRepository, never()).save(any(User.class));
    }
    
    @Test
    void shouldGetUserById() {
        // Given
        Long userId = 1L;
        User user = new User();
        user.setId(userId);
        user.setUsername("johndoe");
        user.setEmail("john@example.com");
        
        when(userRepository.findById(userId)).thenReturn(Optional.of(user));
        
        // When
        Optional<User> result = userService.getUserById(userId);
        
        // Then
        assertThat(result).isPresent();
        assertThat(result.get().getUsername()).isEqualTo("johndoe");
        verify(userRepository).findById(userId);
    }
    
    @Test
    void shouldReturnEmptyWhenUserNotFound() {
        // Given
        Long userId = 1L;
        when(userRepository.findById(userId)).thenReturn(Optional.empty());
        
        // When
        Optional<User> result = userService.getUserById(userId);
        
        // Then
        assertThat(result).isEmpty();
        verify(userRepository).findById(userId);
    }
}
```

### 2. Controller Testing

```java
@WebMvcTest(UserController.class)
class UserControllerTest {
    
    @Autowired
    private MockMvc mockMvc;
    
    @MockBean
    private UserService userService;
    
    @Autowired
    private ObjectMapper objectMapper;
    
    @Test
    void shouldCreateUserSuccessfully() throws Exception {
        // Given
        CreateUserRequest request = new CreateUserRequest();
        request.setUsername("johndoe");
        request.setEmail("john@example.com");
        request.setPassword("password123");
        request.setFirstName("John");
        request.setLastName("Doe");
        
        User user = new User();
        user.setId(1L);
        user.setUsername("johndoe");
        user.setEmail("john@example.com");
        
        when(userService.createUser(any(CreateUserRequest.class))).thenReturn(user);
        
        // When & Then
        mockMvc.perform(post("/api/users")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(request)))
                .andExpect(status().isCreated())
                .andExpect(jsonPath("$.success").value(true))
                .andExpect(jsonPath("$.data.username").value("johndoe"))
                .andExpect(jsonPath("$.data.email").value("john@example.com"));
        
        verify(userService).createUser(any(CreateUserRequest.class));
    }
    
    @Test
    void shouldReturnBadRequestWhenValidationFails() throws Exception {
        // Given
        CreateUserRequest request = new CreateUserRequest();
        request.setUsername(""); // Invalid username
        request.setEmail("invalid-email"); // Invalid email
        request.setPassword("123"); // Too short password
        
        // When & Then
        mockMvc.perform(post("/api/users")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(request)))
                .andExpect(status().isBadRequest())
                .andExpect(jsonPath("$.success").value(false));
        
        verify(userService, never()).createUser(any(CreateUserRequest.class));
    }
    
    @Test
    void shouldGetUserById() throws Exception {
        // Given
        Long userId = 1L;
        User user = new User();
        user.setId(userId);
        user.setUsername("johndoe");
        user.setEmail("john@example.com");
        
        when(userService.getUserById(userId)).thenReturn(Optional.of(user));
        
        // When & Then
        mockMvc.perform(get("/api/users/{id}", userId))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.success").value(true))
                .andExpect(jsonPath("$.data.username").value("johndoe"))
                .andExpect(jsonPath("$.data.email").value("john@example.com"));
        
        verify(userService).getUserById(userId);
    }
    
    @Test
    void shouldReturnNotFoundWhenUserDoesNotExist() throws Exception {
        // Given
        Long userId = 1L;
        when(userService.getUserById(userId)).thenReturn(Optional.empty());
        
        // When & Then
        mockMvc.perform(get("/api/users/{id}", userId))
                .andExpect(status().isNotFound());
        
        verify(userService).getUserById(userId);
    }
}
```

## Integration Testing

### 1. Repository Testing

```java
@DataJpaTest
class UserRepositoryTest {
    
    @Autowired
    private TestEntityManager entityManager;
    
    @Autowired
    private UserRepository userRepository;
    
    @Test
    void shouldFindUserByUsername() {
        // Given
        User user = new User();
        user.setUsername("johndoe");
        user.setEmail("john@example.com");
        user.setPassword("password123");
        user.setFirstName("John");
        user.setLastName("Doe");
        
        entityManager.persistAndFlush(user);
        
        // When
        Optional<User> found = userRepository.findByUsername("johndoe");
        
        // Then
        assertThat(found).isPresent();
        assertThat(found.get().getUsername()).isEqualTo("johndoe");
    }
    
    @Test
    void shouldFindUsersByStatus() {
        // Given
        User activeUser = new User();
        activeUser.setUsername("activeuser");
        activeUser.setEmail("active@example.com");
        activeUser.setPassword("password123");
        activeUser.setFirstName("Active");
        activeUser.setLastName("User");
        activeUser.setStatus(UserStatus.ACTIVE);
        
        User inactiveUser = new User();
        inactiveUser.setUsername("inactiveuser");
        inactiveUser.setEmail("inactive@example.com");
        inactiveUser.setPassword("password123");
        inactiveUser.setFirstName("Inactive");
        inactiveUser.setLastName("User");
        inactiveUser.setStatus(UserStatus.INACTIVE);
        
        entityManager.persistAndFlush(activeUser);
        entityManager.persistAndFlush(inactiveUser);
        
        // When
        List<User> activeUsers = userRepository.findByStatus(UserStatus.ACTIVE);
        
        // Then
        assertThat(activeUsers).hasSize(1);
        assertThat(activeUsers.get(0).getStatus()).isEqualTo(UserStatus.ACTIVE);
    }
}
```

### 2. Full Integration Test

```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@Testcontainers
class UserControllerIntegrationTest {
    
    @Container
    static MySQLContainer<?> mysql = new MySQLContainer<>("mysql:8.0")
            .withDatabaseName("testdb")
            .withUsername("test")
            .withPassword("test");
    
    @Autowired
    private TestRestTemplate restTemplate;
    
    @Autowired
    private UserRepository userRepository;
    
    @DynamicPropertySource
    static void configureProperties(DynamicPropertyRegistry registry) {
        registry.add("spring.datasource.url", mysql::getJdbcUrl);
        registry.add("spring.datasource.username", mysql::getUsername);
        registry.add("spring.datasource.password", mysql::getPassword);
    }
    
    @BeforeEach
    void setUp() {
        userRepository.deleteAll();
    }
    
    @Test
    void shouldCreateAndRetrieveUser() {
        // Given
        CreateUserRequest request = new CreateUserRequest();
        request.setUsername("johndoe");
        request.setEmail("john@example.com");
        request.setPassword("password123");
        request.setFirstName("John");
        request.setLastName("Doe");
        
        // When
        ResponseEntity<ApiResponse<User>> createResponse = restTemplate.postForEntity(
                "/api/users", request, new ParameterizedTypeReference<ApiResponse<User>>() {}
        );
        
        // Then
        assertThat(createResponse.getStatusCode()).isEqualTo(HttpStatus.CREATED);
        assertThat(createResponse.getBody().getSuccess()).isTrue();
        assertThat(createResponse.getBody().getData().getUsername()).isEqualTo("johndoe");
        
        // When - Get user
        Long userId = createResponse.getBody().getData().getId();
        ResponseEntity<ApiResponse<User>> getResponse = restTemplate.getForEntity(
                "/api/users/" + userId, new ParameterizedTypeReference<ApiResponse<User>>() {}
        );
        
        // Then
        assertThat(getResponse.getStatusCode()).isEqualTo(HttpStatus.OK);
        assertThat(getResponse.getBody().getData().getUsername()).isEqualTo("johndoe");
    }
    
    @Test
    void shouldReturnAllUsers() {
        // Given
        User user1 = new User();
        user1.setUsername("user1");
        user1.setEmail("user1@example.com");
        user1.setPassword("password123");
        user1.setFirstName("User");
        user1.setLastName("One");
        
        User user2 = new User();
        user2.setUsername("user2");
        user2.setEmail("user2@example.com");
        user2.setPassword("password123");
        user2.setFirstName("User");
        user2.setLastName("Two");
        
        userRepository.saveAll(Arrays.asList(user1, user2));
        
        // When
        ResponseEntity<ApiResponse<List<User>>> response = restTemplate.exchange(
                "/api/users", HttpMethod.GET, null,
                new ParameterizedTypeReference<ApiResponse<List<User>>>() {}
        );
        
        // Then
        assertThat(response.getStatusCode()).isEqualTo(HttpStatus.OK);
        assertThat(response.getBody().getData()).hasSize(2);
    }
}
```

## Testing với TestContainers

### 1. Database Testing

```java
@SpringBootTest
@Testcontainers
class DatabaseIntegrationTest {
    
    @Container
    static MySQLContainer<?> mysql = new MySQLContainer<>("mysql:8.0")
            .withDatabaseName("testdb")
            .withUsername("test")
            .withPassword("test");
    
    @DynamicPropertySource
    static void configureProperties(DynamicPropertyRegistry registry) {
        registry.add("spring.datasource.url", mysql::getJdbcUrl);
        registry.add("spring.datasource.username", mysql::getUsername);
        registry.add("spring.datasource.password", mysql::getPassword);
    }
    
    @Test
    void shouldConnectToDatabase() {
        // Test database connection
        assertThat(mysql.isRunning()).isTrue();
    }
}
```

### 2. Redis Testing

```java
@SpringBootTest
@Testcontainers
class RedisIntegrationTest {
    
    @Container
    static GenericContainer<?> redis = new GenericContainer<>("redis:7-alpine")
            .withExposedPorts(6379);
    
    @DynamicPropertySource
    static void configureProperties(DynamicPropertyRegistry registry) {
        registry.add("spring.redis.host", redis::getHost);
        registry.add("spring.redis.port", () -> redis.getMappedPort(6379));
    }
    
    @Test
    void shouldConnectToRedis() {
        assertThat(redis.isRunning()).isTrue();
    }
}
```

## Testing Configuration

### 1. Test Configuration

```java
@TestConfiguration
public class TestConfig {
    
    @Bean
    @Primary
    public PasswordEncoder testPasswordEncoder() {
        return new BCryptPasswordEncoder();
    }
    
    @Bean
    @Primary
    public Clock testClock() {
        return Clock.fixed(Instant.parse("2024-01-01T00:00:00Z"), ZoneOffset.UTC);
    }
}
```

### 2. Test Properties

```properties
# application-test.properties
spring.datasource.url=jdbc:h2:mem:testdb
spring.datasource.driver-class-name=org.h2.Driver
spring.jpa.hibernate.ddl-auto=create-drop
spring.jpa.show-sql=true
logging.level.org.springframework.web=DEBUG
```

## Performance Testing

### 1. Load Testing

```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
class PerformanceTest {
    
    @Autowired
    private TestRestTemplate restTemplate;
    
    @Test
    void shouldHandleConcurrentRequests() throws InterruptedException {
        int numberOfThreads = 10;
        int requestsPerThread = 100;
        CountDownLatch latch = new CountDownLatch(numberOfThreads);
        AtomicInteger successCount = new AtomicInteger(0);
        AtomicInteger errorCount = new AtomicInteger(0);
        
        for (int i = 0; i < numberOfThreads; i++) {
            new Thread(() -> {
                try {
                    for (int j = 0; j < requestsPerThread; j++) {
                        try {
                            ResponseEntity<String> response = restTemplate.getForEntity(
                                    "/api/users", String.class);
                            if (response.getStatusCode().is2xxSuccessful()) {
                                successCount.incrementAndGet();
                            } else {
                                errorCount.incrementAndGet();
                            }
                        } catch (Exception e) {
                            errorCount.incrementAndGet();
                        }
                    }
                } finally {
                    latch.countDown();
                }
            }).start();
        }
        
        latch.await(30, TimeUnit.SECONDS);
        
        int totalRequests = numberOfThreads * requestsPerThread;
        assertThat(successCount.get()).isGreaterThan(totalRequests * 0.9); // 90% success rate
        assertThat(errorCount.get()).isLessThan(totalRequests * 0.1); // Less than 10% errors
    }
}
```

## Test Coverage

### 1. JaCoCo Configuration

```xml
<plugin>
    <groupId>org.jacoco</groupId>
    <artifactId>jacoco-maven-plugin</artifactId>
    <version>0.8.8</version>
    <executions>
        <execution>
            <goals>
                <goal>prepare-agent</goal>
            </goals>
        </execution>
        <execution>
            <id>report</id>
            <phase>test</phase>
            <goals>
                <goal>report</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

### 2. Coverage Goals

```xml
<plugin>
    <groupId>org.jacoco</groupId>
    <artifactId>jacoco-maven-plugin</artifactId>
    <configuration>
        <rules>
            <rule>
                <element>BUNDLE</element>
                <limits>
                    <limit>
                        <counter>LINE</counter>
                        <value>COVEREDRATIO</value>
                        <minimum>0.80</minimum>
                    </limit>
                </limits>
            </rule>
        </rules>
    </configuration>
</plugin>
```

## Best Practices

### 1. Test Naming

```java
// ✅ Good naming
@Test
void shouldReturnUserWhenValidIdProvided() { }

@Test
void shouldThrowExceptionWhenUserNotFound() { }

// ❌ Bad naming
@Test
void test1() { }

@Test
void testUser() { }
```

### 2. Test Structure (AAA Pattern)

```java
@Test
void shouldCreateUserSuccessfully() {
    // Arrange - Given
    CreateUserRequest request = new CreateUserRequest();
    request.setUsername("johndoe");
    // ... setup data
    
    when(userRepository.existsByUsername("johndoe")).thenReturn(false);
    // ... setup mocks
    
    // Act - When
    User result = userService.createUser(request);
    
    // Assert - Then
    assertThat(result).isNotNull();
    assertThat(result.getUsername()).isEqualTo("johndoe");
    // ... verify results
}
```

### 3. Test Isolation

```java
@BeforeEach
void setUp() {
    // Reset mocks
    reset(userRepository, passwordEncoder);
    
    // Clear database
    userRepository.deleteAll();
}
```

## Kết Luận

Testing trong Spring Boot cung cấp một bộ công cụ mạnh mẽ để đảm bảo chất lượng code. Với những kiến thức này, bạn có thể:

- Viết unit test hiệu quả với Mockito
- Thực hiện integration test với TestContainers
- Test performance và load
- Đo lường test coverage
- Áp dụng best practices

Trong bài viết tiếp theo, tôi sẽ hướng dẫn về Docker và deployment! 🚀


