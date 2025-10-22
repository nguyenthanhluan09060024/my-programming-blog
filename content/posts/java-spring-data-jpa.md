---
title: "Spring Data JPA - Làm Việc Với Database Hiệu Quả"
date: 2024-01-08T15:00:00+07:00
draft: false
tags: ['java', 'spring-data-jpa', 'database', 'orm']
description: "Hướng dẫn chi tiết về Spring Data JPA để làm việc với database một cách hiệu quả"
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: true
ShowCodeCopyButtons: true
toc: true
---

# Spring Data JPA - Làm Việc Với Database Hiệu Quả

Spring Data JPA giúp đơn giản hóa việc làm việc với database trong ứng dụng Spring Boot. Trong bài viết này, tôi sẽ hướng dẫn bạn cách sử dụng Spring Data JPA một cách hiệu quả.

## Spring Data JPA là gì?

Spring Data JPA là một phần của Spring Data project, cung cấp một abstraction layer trên JPA (Java Persistence API) để đơn giản hóa việc phát triển các ứng dụng sử dụng database.

### Ưu điểm:
- **Giảm boilerplate code**: Tự động generate implementation
- **Query methods**: Tạo queries từ method names
- **Pagination**: Hỗ trợ phân trang built-in
- **Auditing**: Tự động track create/update timestamps
- **Caching**: Tích hợp với Spring Cache

## Cài Đặt và Cấu Hình

### 1. Dependencies

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-validation</artifactId>
    </dependency>
</dependencies>
```

### 2. Application Properties

```properties
# Database Configuration
spring.datasource.url=jdbc:mysql://localhost:3306/myapp?useSSL=false&serverTimezone=UTC
spring.datasource.username=root
spring.datasource.password=password
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver

# JPA Configuration
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL8Dialect
spring.jpa.properties.hibernate.format_sql=true

# Connection Pool
spring.datasource.hikari.maximum-pool-size=20
spring.datasource.hikari.minimum-idle=5
spring.datasource.hikari.idle-timeout=300000
spring.datasource.hikari.max-lifetime=1200000
```

## Entity Mapping

### 1. Basic Entity

```java
@Entity
@Table(name = "users")
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(name = "username", unique = true, nullable = false)
    private String username;
    
    @Column(name = "email", unique = true, nullable = false)
    private String email;
    
    @Column(name = "first_name")
    private String firstName;
    
    @Column(name = "last_name")
    private String lastName;
    
    @Enumerated(EnumType.STRING)
    @Column(name = "status")
    private UserStatus status = UserStatus.ACTIVE;
    
    @CreationTimestamp
    @Column(name = "created_at", updatable = false)
    private LocalDateTime createdAt;
    
    @UpdateTimestamp
    @Column(name = "updated_at")
    private LocalDateTime updatedAt;
    
    // Constructors, getters, setters
}

public enum UserStatus {
    ACTIVE, INACTIVE, SUSPENDED
}
```

### 2. Relationship Mapping

```java
@Entity
@Table(name = "orders")
public class Order {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "user_id", nullable = false)
    private User user;
    
    @OneToMany(mappedBy = "order", cascade = CascadeType.ALL, fetch = FetchType.LAZY)
    private List<OrderItem> items = new ArrayList<>();
    
    @Column(name = "total_amount", precision = 10, scale = 2)
    private BigDecimal totalAmount;
    
    @Enumerated(EnumType.STRING)
    @Column(name = "status")
    private OrderStatus status = OrderStatus.PENDING;
    
    @CreationTimestamp
    @Column(name = "created_at", updatable = false)
    private LocalDateTime createdAt;
    
    // Constructors, getters, setters
}

@Entity
@Table(name = "order_items")
public class OrderItem {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "order_id", nullable = false)
    private Order order;
    
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "product_id", nullable = false)
    private Product product;
    
    @Column(name = "quantity", nullable = false)
    private Integer quantity;
    
    @Column(name = "price", precision = 10, scale = 2, nullable = false)
    private BigDecimal price;
    
    // Constructors, getters, setters
}
```

## Repository Layer

### 1. Basic Repository

```java
@Repository
public interface UserRepository extends JpaRepository<User, Long> {
    // Query methods
    Optional<User> findByUsername(String username);
    Optional<User> findByEmail(String email);
    List<User> findByStatus(UserStatus status);
    List<User> findByFirstNameContainingIgnoreCase(String firstName);
    List<User> findByCreatedAtBetween(LocalDateTime start, LocalDateTime end);
    
    // Custom queries
    @Query("SELECT u FROM User u WHERE u.status = :status AND u.createdAt >= :date")
    List<User> findActiveUsersAfterDate(@Param("status") UserStatus status, 
                                       @Param("date") LocalDateTime date);
    
    @Query(value = "SELECT * FROM users WHERE first_name LIKE %:name%", nativeQuery = true)
    List<User> findUsersByNameNative(@Param("name") String name);
    
    // Exists queries
    boolean existsByUsername(String username);
    boolean existsByEmail(String email);
    
    // Count queries
    long countByStatus(UserStatus status);
    
    // Delete queries
    void deleteByStatus(UserStatus status);
}
```

### 2. Custom Repository Implementation

```java
public interface UserRepositoryCustom {
    List<User> findUsersWithComplexCriteria(UserSearchCriteria criteria);
    Page<User> findUsersWithPagination(UserSearchCriteria criteria, Pageable pageable);
}

@Repository
public class UserRepositoryImpl implements UserRepositoryCustom {
    
    @PersistenceContext
    private EntityManager entityManager;
    
    @Override
    public List<User> findUsersWithComplexCriteria(UserSearchCriteria criteria) {
        CriteriaBuilder cb = entityManager.getCriteriaBuilder();
        CriteriaQuery<User> query = cb.createQuery(User.class);
        Root<User> user = query.from(User.class);
        
        List<Predicate> predicates = new ArrayList<>();
        
        if (criteria.getUsername() != null) {
            predicates.add(cb.like(user.get("username"), "%" + criteria.getUsername() + "%"));
        }
        
        if (criteria.getStatus() != null) {
            predicates.add(cb.equal(user.get("status"), criteria.getStatus()));
        }
        
        if (criteria.getStartDate() != null) {
            predicates.add(cb.greaterThanOrEqualTo(user.get("createdAt"), criteria.getStartDate()));
        }
        
        if (criteria.getEndDate() != null) {
            predicates.add(cb.lessThanOrEqualTo(user.get("createdAt"), criteria.getEndDate()));
        }
        
        query.where(predicates.toArray(new Predicate[0]));
        
        return entityManager.createQuery(query).getResultList();
    }
    
    @Override
    public Page<User> findUsersWithPagination(UserSearchCriteria criteria, Pageable pageable) {
        CriteriaBuilder cb = entityManager.getCriteriaBuilder();
        CriteriaQuery<User> query = cb.createQuery(User.class);
        Root<User> user = query.from(User.class);
        
        // Add predicates...
        
        query.where(predicates.toArray(new Predicate[0]));
        
        List<User> users = entityManager.createQuery(query)
            .setFirstResult((int) pageable.getOffset())
            .setMaxResults(pageable.getPageSize())
            .getResultList();
        
        // Count query
        CriteriaQuery<Long> countQuery = cb.createQuery(Long.class);
        countQuery.select(cb.count(countQuery.from(User.class)));
        countQuery.where(predicates.toArray(new Predicate[0]));
        
        Long total = entityManager.createQuery(countQuery).getSingleResult();
        
        return new PageImpl<>(users, pageable, total);
    }
}
```

## Service Layer

### 1. Basic Service

```java
@Service
@Transactional
public class UserService {
    
    @Autowired
    private UserRepository userRepository;
    
    public List<User> getAllUsers() {
        return userRepository.findAll();
    }
    
    public Optional<User> getUserById(Long id) {
        return userRepository.findById(id);
    }
    
    public User createUser(CreateUserRequest request) {
        if (userRepository.existsByUsername(request.getUsername())) {
            throw new RuntimeException("Username already exists");
        }
        
        if (userRepository.existsByEmail(request.getEmail())) {
            throw new RuntimeException("Email already exists");
        }
        
        User user = new User();
        user.setUsername(request.getUsername());
        user.setEmail(request.getEmail());
        user.setFirstName(request.getFirstName());
        user.setLastName(request.getLastName());
        
        return userRepository.save(user);
    }
    
    public User updateUser(Long id, UpdateUserRequest request) {
        User user = userRepository.findById(id)
            .orElseThrow(() -> new RuntimeException("User not found"));
        
        user.setFirstName(request.getFirstName());
        user.setLastName(request.getLastName());
        user.setEmail(request.getEmail());
        
        return userRepository.save(user);
    }
    
    public void deleteUser(Long id) {
        userRepository.deleteById(id);
    }
}
```

### 2. Advanced Service với Pagination

```java
@Service
@Transactional
public class UserService {
    
    @Autowired
    private UserRepository userRepository;
    
    public Page<User> getUsers(UserSearchCriteria criteria, Pageable pageable) {
        return userRepository.findUsersWithPagination(criteria, pageable);
    }
    
    public List<User> searchUsers(UserSearchCriteria criteria) {
        return userRepository.findUsersWithComplexCriteria(criteria);
    }
    
    @Transactional(readOnly = true)
    public UserStatistics getUserStatistics() {
        long totalUsers = userRepository.count();
        long activeUsers = userRepository.countByStatus(UserStatus.ACTIVE);
        long inactiveUsers = userRepository.countByStatus(UserStatus.INACTIVE);
        
        return new UserStatistics(totalUsers, activeUsers, inactiveUsers);
    }
}
```

## Pagination và Sorting

### 1. Controller với Pagination

```java
@RestController
@RequestMapping("/api/users")
public class UserController {
    
    @Autowired
    private UserService userService;
    
    @GetMapping
    public ResponseEntity<Page<User>> getUsers(
            @RequestParam(defaultValue = "0") int page,
            @RequestParam(defaultValue = "10") int size,
            @RequestParam(defaultValue = "id") String sortBy,
            @RequestParam(defaultValue = "asc") String sortDir,
            @RequestParam(required = false) String search,
            @RequestParam(required = false) UserStatus status) {
        
        Sort sort = sortDir.equalsIgnoreCase("desc") 
            ? Sort.by(sortBy).descending() 
            : Sort.by(sortBy).ascending();
        
        Pageable pageable = PageRequest.of(page, size, sort);
        
        UserSearchCriteria criteria = new UserSearchCriteria();
        criteria.setSearch(search);
        criteria.setStatus(status);
        
        Page<User> users = userService.getUsers(criteria, pageable);
        
        return ResponseEntity.ok(users);
    }
}
```

## Caching

### 1. Enable Caching

```java
@SpringBootApplication
@EnableCaching
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

### 2. Cache Configuration

```java
@Configuration
@EnableCaching
public class CacheConfig {
    
    @Bean
    public CacheManager cacheManager() {
        CaffeineCacheManager cacheManager = new CaffeineCacheManager();
        cacheManager.setCaffeine(Caffeine.newBuilder()
            .maximumSize(1000)
            .expireAfterWrite(10, TimeUnit.MINUTES));
        return cacheManager;
    }
}
```

### 3. Caching trong Service

```java
@Service
public class UserService {
    
    @Cacheable(value = "users", key = "#id")
    public Optional<User> getUserById(Long id) {
        return userRepository.findById(id);
    }
    
    @CacheEvict(value = "users", key = "#user.id")
    public User updateUser(User user) {
        return userRepository.save(user);
    }
    
    @CacheEvict(value = "users", allEntries = true)
    public void clearCache() {
        // Clear all cache entries
    }
}
```

## Auditing

### 1. Enable Auditing

```java
@SpringBootApplication
@EnableJpaAuditing
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

### 2. Auditing Entity

```java
@Entity
@EntityListeners(AuditingEntityListener.class)
public class User {
    // ... other fields
    
    @CreatedBy
    @Column(name = "created_by")
    private String createdBy;
    
    @LastModifiedBy
    @Column(name = "last_modified_by")
    private String lastModifiedBy;
    
    @CreationTimestamp
    @Column(name = "created_at", updatable = false)
    private LocalDateTime createdAt;
    
    @UpdateTimestamp
    @Column(name = "updated_at")
    private LocalDateTime updatedAt;
}
```

### 3. Auditor Provider

```java
@Component
public class AuditorProvider implements AuditorAware<String> {
    
    @Override
    public Optional<String> getCurrentAuditor() {
        // Get current user from security context
        Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
        if (authentication != null && authentication.isAuthenticated()) {
            return Optional.of(authentication.getName());
        }
        return Optional.of("system");
    }
}
```

## Batch Operations

### 1. Batch Insert

```java
@Service
public class UserService {
    
    @Autowired
    private UserRepository userRepository;
    
    @Transactional
    public void batchCreateUsers(List<CreateUserRequest> requests) {
        List<User> users = requests.stream()
            .map(this::mapToUser)
            .collect(Collectors.toList());
        
        userRepository.saveAll(users);
    }
    
    @Transactional
    public void batchUpdateUsers(List<UpdateUserRequest> requests) {
        List<User> users = requests.stream()
            .map(this::mapToUser)
            .collect(Collectors.toList());
        
        userRepository.saveAll(users);
    }
}
```

### 2. Batch Delete

```java
@Repository
public interface UserRepository extends JpaRepository<User, Long> {
    
    @Modifying
    @Query("DELETE FROM User u WHERE u.status = :status")
    void deleteByStatus(@Param("status") UserStatus status);
    
    @Modifying
    @Query("DELETE FROM User u WHERE u.createdAt < :date")
    void deleteUsersOlderThan(@Param("date") LocalDateTime date);
}
```

## Performance Optimization

### 1. Lazy Loading

```java
@Entity
public class User {
    @OneToMany(mappedBy = "user", fetch = FetchType.LAZY)
    private List<Order> orders = new ArrayList<>();
}
```

### 2. Eager Loading với @EntityGraph

```java
@EntityGraph(attributePaths = {"orders", "orders.items"})
@Query("SELECT u FROM User u WHERE u.id = :id")
Optional<User> findByIdWithOrders(@Param("id") Long id);
```

### 3. Projection

```java
public interface UserProjection {
    String getUsername();
    String getEmail();
    String getFirstName();
    String getLastName();
}

@Repository
public interface UserRepository extends JpaRepository<User, Long> {
    List<UserProjection> findByStatus(UserStatus status);
}
```

## Kết Luận

Spring Data JPA cung cấp một cách mạnh mẽ và hiệu quả để làm việc với database. Với những kiến thức này, bạn có thể:

- Tạo entities và relationships
- Sử dụng query methods và custom queries
- Implement pagination và sorting
- Sử dụng caching để tối ưu performance
- Thực hiện batch operations
- Tối ưu hóa queries

Trong bài viết tiếp theo, tôi sẽ hướng dẫn về testing với Spring Boot! 🚀


