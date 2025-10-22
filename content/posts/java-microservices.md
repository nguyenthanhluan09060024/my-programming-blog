---
title: "Xây Dựng Microservices Với Spring Boot"
date: 2024-01-12T16:00:00+07:00
draft: false
tags: ['java', 'microservices', 'spring-boot', 'architecture']
description: "Hướng dẫn xây dựng kiến trúc microservices với Spring Boot và các công cụ liên quan"
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: true
ShowCodeCopyButtons: true
toc: true
---

# Xây Dựng Microservices Với Spring Boot

Microservices là một kiến trúc phần mềm cho phép phát triển ứng dụng như một tập hợp các dịch vụ nhỏ, độc lập. Trong bài viết này, tôi sẽ hướng dẫn bạn cách xây dựng microservices với Spring Boot.

## Microservices là gì?

Microservices là một phương pháp phát triển phần mềm trong đó một ứng dụng lớn được xây dựng như một tập hợp các dịch vụ nhỏ, có thể triển khai độc lập.

### Ưu điểm:
- **Scalability**: Có thể scale từng service riêng biệt
- **Technology Diversity**: Mỗi service có thể sử dụng công nghệ khác nhau
- **Fault Isolation**: Lỗi ở một service không ảnh hưởng đến toàn bộ hệ thống
- **Team Independence**: Các team có thể phát triển độc lập

### Nhược điểm:
- **Complexity**: Tăng độ phức tạp của hệ thống
- **Network Latency**: Giao tiếp qua network
- **Data Consistency**: Khó đảm bảo consistency
- **Testing**: Khó test toàn bộ hệ thống

## Kiến Trúc Microservices

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   API Gateway   │    │   Load Balancer │    │   Service Mesh  │
└─────────────────┘    └─────────────────┘    └─────────────────┘
         │                       │                       │
    ┌────┴────┐              ┌────┴────┐              ┌────┴────┐
    │         │              │         │              │         │
┌───▼───┐ ┌───▼───┐    ┌────▼────┐ ┌──▼───┐    ┌────▼────┐ ┌──▼───┐
│ User  │ │ Order │    │Product  │ │Payment│    │Notification│ │Logging│
│Service│ │Service│    │Service  │ │Service│    │Service     │ │Service│
└───────┘ └───────┘    └─────────┘ └──────┘    └────────────┘ └──────┘
    │         │              │         │              │         │
    └─────────┼──────────────┼─────────┼──────────────┼─────────┘
              │              │         │              │
         ┌────▼────┐    ┌────▼────┐ ┌──▼───┐    ┌────▼────┐
         │Database │    │Database │ │Database│    │Message │
         │(User)   │    │(Product)│ │(Order) │    │Queue   │
         └─────────┘    └─────────┘ └───────┘    └─────────┘
```

## Tạo User Service

### 1. Cấu trúc dự án

```
user-service/
├── src/
│   ├── main/
│   │   ├── java/
│   │   │   └── com/example/userservice/
│   │   │       ├── UserServiceApplication.java
│   │   │       ├── controller/
│   │   │       ├── service/
│   │   │       ├── repository/
│   │   │       ├── model/
│   │   │       └── config/
│   │   └── resources/
│   │       └── application.yml
│   └── test/
└── pom.xml
```

### 2. Dependencies

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-validation</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-openfeign</artifactId>
    </dependency>
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
    </dependency>
</dependencies>
```

### 3. User Entity

```java
@Entity
@Table(name = "users")
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(unique = true, nullable = false)
    private String username;
    
    @Column(nullable = false)
    private String email;
    
    @Column(nullable = false)
    private String firstName;
    
    @Column(nullable = false)
    private String lastName;
    
    @Enumerated(EnumType.STRING)
    private UserStatus status = UserStatus.ACTIVE;
    
    @CreationTimestamp
    private LocalDateTime createdAt;
    
    @UpdateTimestamp
    private LocalDateTime updatedAt;
    
    // Constructors, getters, setters
}

public enum UserStatus {
    ACTIVE, INACTIVE, SUSPENDED
}
```

### 4. User Repository

```java
@Repository
public interface UserRepository extends JpaRepository<User, Long> {
    Optional<User> findByUsername(String username);
    Optional<User> findByEmail(String email);
    List<User> findByStatus(UserStatus status);
    boolean existsByUsername(String username);
    boolean existsByEmail(String email);
}
```

### 5. User Service

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
    
    public Optional<User> getUserByUsername(String username) {
        return userRepository.findByUsername(username);
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

### 6. User Controller

```java
@RestController
@RequestMapping("/api/users")
@Validated
public class UserController {
    
    @Autowired
    private UserService userService;
    
    @GetMapping
    public ResponseEntity<List<User>> getAllUsers() {
        List<User> users = userService.getAllUsers();
        return ResponseEntity.ok(users);
    }
    
    @GetMapping("/{id}")
    public ResponseEntity<User> getUserById(@PathVariable Long id) {
        Optional<User> user = userService.getUserById(id);
        return user.map(ResponseEntity::ok)
                  .orElse(ResponseEntity.notFound().build());
    }
    
    @PostMapping
    public ResponseEntity<User> createUser(@Valid @RequestBody CreateUserRequest request) {
        User user = userService.createUser(request);
        return ResponseEntity.status(HttpStatus.CREATED).body(user);
    }
    
    @PutMapping("/{id}")
    public ResponseEntity<User> updateUser(@PathVariable Long id, 
                                         @Valid @RequestBody UpdateUserRequest request) {
        User user = userService.updateUser(id, request);
        return ResponseEntity.ok(user);
    }
    
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteUser(@PathVariable Long id) {
        userService.deleteUser(id);
        return ResponseEntity.noContent().build();
    }
}
```

## Tạo Order Service

### 1. Order Entity

```java
@Entity
@Table(name = "orders")
public class Order {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(nullable = false)
    private Long userId;
    
    @Column(nullable = false)
    private BigDecimal totalAmount;
    
    @Enumerated(EnumType.STRING)
    private OrderStatus status = OrderStatus.PENDING;
    
    @CreationTimestamp
    private LocalDateTime createdAt;
    
    @OneToMany(mappedBy = "order", cascade = CascadeType.ALL)
    private List<OrderItem> items = new ArrayList<>();
    
    // Constructors, getters, setters
}

@Entity
@Table(name = "order_items")
public class OrderItem {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @ManyToOne
    @JoinColumn(name = "order_id")
    private Order order;
    
    @Column(nullable = false)
    private Long productId;
    
    @Column(nullable = false)
    private Integer quantity;
    
    @Column(nullable = false)
    private BigDecimal price;
    
    // Constructors, getters, setters
}

public enum OrderStatus {
    PENDING, CONFIRMED, SHIPPED, DELIVERED, CANCELLED
}
```

### 2. Feign Client cho User Service

```java
@FeignClient(name = "user-service")
public interface UserServiceClient {
    
    @GetMapping("/api/users/{id}")
    User getUserById(@PathVariable("id") Long id);
    
    @GetMapping("/api/users/username/{username}")
    User getUserByUsername(@PathVariable("username") String username);
}
```

### 3. Order Service

```java
@Service
@Transactional
public class OrderService {
    
    @Autowired
    private OrderRepository orderRepository;
    
    @Autowired
    private UserServiceClient userServiceClient;
    
    public List<Order> getAllOrders() {
        return orderRepository.findAll();
    }
    
    public Optional<Order> getOrderById(Long id) {
        return orderRepository.findById(id);
    }
    
    public Order createOrder(CreateOrderRequest request) {
        // Validate user exists
        try {
            User user = userServiceClient.getUserById(request.getUserId());
            if (user == null) {
                throw new RuntimeException("User not found");
            }
        } catch (Exception e) {
            throw new RuntimeException("User service unavailable");
        }
        
        Order order = new Order();
        order.setUserId(request.getUserId());
        order.setStatus(OrderStatus.PENDING);
        
        BigDecimal totalAmount = BigDecimal.ZERO;
        for (OrderItemRequest itemRequest : request.getItems()) {
            OrderItem item = new OrderItem();
            item.setProductId(itemRequest.getProductId());
            item.setQuantity(itemRequest.getQuantity());
            item.setPrice(itemRequest.getPrice());
            item.setOrder(order);
            
            order.getItems().add(item);
            totalAmount = totalAmount.add(item.getPrice().multiply(BigDecimal.valueOf(item.getQuantity())));
        }
        
        order.setTotalAmount(totalAmount);
        
        return orderRepository.save(order);
    }
    
    public Order updateOrderStatus(Long id, OrderStatus status) {
        Order order = orderRepository.findById(id)
            .orElseThrow(() -> new RuntimeException("Order not found"));
        
        order.setStatus(status);
        return orderRepository.save(order);
    }
}
```

## API Gateway với Spring Cloud Gateway

### 1. Gateway Dependencies

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-gateway</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
    </dependency>
</dependencies>
```

### 2. Gateway Configuration

```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: user-service
          uri: lb://user-service
          predicates:
            - Path=/api/users/**
        - id: order-service
          uri: lb://order-service
          predicates:
            - Path=/api/orders/**
        - id: product-service
          uri: lb://product-service
          predicates:
            - Path=/api/products/**
      globalcors:
        cors-configurations:
          '[/**]':
            allowedOrigins: "*"
            allowedMethods: "*"
            allowedHeaders: "*"
```

## Service Discovery với Eureka

### 1. Eureka Server

```java
@SpringBootApplication
@EnableEurekaServer
public class EurekaServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(EurekaServerApplication.class, args);
    }
}
```

### 2. Eureka Client Configuration

```yaml
eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/
  instance:
    prefer-ip-address: true
    instance-id: ${spring.application.name}:${spring.application.instance_id:${random.value}}
```

## Inter-Service Communication

### 1. Synchronous Communication với Feign

```java
@FeignClient(name = "product-service", fallback = ProductServiceFallback.class)
public interface ProductServiceClient {
    
    @GetMapping("/api/products/{id}")
    Product getProductById(@PathVariable("id") Long id);
    
    @PostMapping("/api/products/check-availability")
    boolean checkAvailability(@RequestBody CheckAvailabilityRequest request);
}

@Component
public class ProductServiceFallback implements ProductServiceClient {
    
    @Override
    public Product getProductById(Long id) {
        return new Product(); // Default product
    }
    
    @Override
    public boolean checkAvailability(CheckAvailabilityRequest request) {
        return false; // Default to not available
    }
}
```

### 2. Asynchronous Communication với Message Queue

```java
@Component
public class OrderEventPublisher {
    
    @Autowired
    private RabbitTemplate rabbitTemplate;
    
    public void publishOrderCreated(Order order) {
        OrderCreatedEvent event = new OrderCreatedEvent();
        event.setOrderId(order.getId());
        event.setUserId(order.getUserId());
        event.setTotalAmount(order.getTotalAmount());
        
        rabbitTemplate.convertAndSend("order.exchange", "order.created", event);
    }
}

@RabbitListener(queues = "order.created.queue")
public class OrderEventListener {
    
    @Autowired
    private NotificationService notificationService;
    
    public void handleOrderCreated(OrderCreatedEvent event) {
        notificationService.sendOrderConfirmation(event.getUserId(), event.getOrderId());
    }
}
```

## Database per Service

### 1. User Service Database

```yaml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/user_db
    username: user_service
    password: password
  jpa:
    hibernate:
      ddl-auto: update
    properties:
      hibernate:
        dialect: org.hibernate.dialect.MySQL8Dialect
```

### 2. Order Service Database

```yaml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/order_db
    username: order_service
    password: password
  jpa:
    hibernate:
      ddl-auto: update
    properties:
      hibernate:
        dialect: org.hibernate.dialect.MySQL8Dialect
```

## Monitoring và Logging

### 1. Spring Boot Actuator

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

### 2. Micrometer với Prometheus

```xml
<dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-registry-prometheus</artifactId>
</dependency>
```

### 3. Distributed Tracing với Sleuth

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-sleuth</artifactId>
</dependency>
```

## Kết Luận

Microservices với Spring Boot cung cấp một cách mạnh mẽ để xây dựng các ứng dụng phân tán. Với những kiến thức này, bạn có thể:

- Tạo các microservices độc lập
- Sử dụng API Gateway để routing
- Implement service discovery
- Xử lý inter-service communication
- Quản lý database per service
- Monitor và trace các services

Trong bài viết tiếp theo, tôi sẽ hướng dẫn về Docker và Kubernetes để deploy microservices! 🚀


