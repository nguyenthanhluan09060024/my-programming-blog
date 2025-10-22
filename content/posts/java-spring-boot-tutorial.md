---
title: "Hướng Dẫn Spring Boot Cho Người Mới Bắt Đầu"
date: 2024-01-20T10:00:00+07:00
draft: false
tags: ['java', 'spring-boot', 'tutorial', 'backend']
description: "Hướng dẫn chi tiết cách xây dựng ứng dụng web với Spring Boot từ cơ bản đến nâng cao"
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: true
ShowCodeCopyButtons: true
toc: true
---

# Hướng Dẫn Spring Boot Cho Người Mới Bắt Đầu

Spring Boot là một framework mạnh mẽ giúp phát triển ứng dụng Java một cách nhanh chóng và dễ dàng. Trong bài viết này, tôi sẽ hướng dẫn bạn từng bước để tạo ra ứng dụng web đầu tiên với Spring Boot.

## Tại Sao Chọn Spring Boot?

- **Auto-configuration**: Tự động cấu hình các thành phần cần thiết
- **Embedded Server**: Có sẵn Tomcat, Jetty hoặc Undertow
- **Production Ready**: Có sẵn các tính năng monitoring và metrics
- **Microservices**: Hỗ trợ tốt cho kiến trúc microservices

## Tạo Dự Án Spring Boot

### 1. Sử dụng Spring Initializr

Truy cập [start.spring.io](https://start.spring.io) và chọn:
- **Project**: Maven
- **Language**: Java
- **Spring Boot**: 3.2.0
- **Dependencies**: Spring Web, Spring Data JPA, H2 Database

### 2. Cấu Trúc Dự Án

```
src/
├── main/
│   ├── java/
│   │   └── com/example/demo/
│   │       ├── DemoApplication.java
│   │       ├── controller/
│   │       ├── service/
│   │       ├── repository/
│   │       └── model/
│   └── resources/
│       ├── application.properties
│       └── static/
└── test/
```

## Xây Dựng REST API Cơ Bản

### 1. Tạo Model

```java
@Entity
@Table(name = "users")
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(nullable = false)
    private String name;
    
    @Column(unique = true, nullable = false)
    private String email;
    
    // Constructors, getters, setters
    public User() {}
    
    public User(String name, String email) {
        this.name = name;
        this.email = email;
    }
    
    // Getters and Setters
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }
    
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
    
    public String getEmail() { return email; }
    public void setEmail(String email) { this.email = email; }
}
```

### 2. Tạo Repository

```java
@Repository
public interface UserRepository extends JpaRepository<User, Long> {
    Optional<User> findByEmail(String email);
    List<User> findByNameContaining(String name);
}
```

### 3. Tạo Service

```java
@Service
public class UserService {
    
    @Autowired
    private UserRepository userRepository;
    
    public List<User> getAllUsers() {
        return userRepository.findAll();
    }
    
    public Optional<User> getUserById(Long id) {
        return userRepository.findById(id);
    }
    
    public User createUser(User user) {
        return userRepository.save(user);
    }
    
    public User updateUser(Long id, User userDetails) {
        User user = userRepository.findById(id)
            .orElseThrow(() -> new RuntimeException("User not found"));
        
        user.setName(userDetails.getName());
        user.setEmail(userDetails.getEmail());
        
        return userRepository.save(user);
    }
    
    public void deleteUser(Long id) {
        userRepository.deleteById(id);
    }
}
```

### 4. Tạo Controller

```java
@RestController
@RequestMapping("/api/users")
@CrossOrigin(origins = "*")
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
    public ResponseEntity<User> createUser(@RequestBody User user) {
        User createdUser = userService.createUser(user);
        return ResponseEntity.status(HttpStatus.CREATED).body(createdUser);
    }
    
    @PutMapping("/{id}")
    public ResponseEntity<User> updateUser(@PathVariable Long id, @RequestBody User user) {
        User updatedUser = userService.updateUser(id, user);
        return ResponseEntity.ok(updatedUser);
    }
    
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteUser(@PathVariable Long id) {
        userService.deleteUser(id);
        return ResponseEntity.noContent().build();
    }
}
```

## Cấu Hình Database

### application.properties

```properties
# Database Configuration
spring.datasource.url=jdbc:h2:mem:testdb
spring.datasource.driverClassName=org.h2.Driver
spring.datasource.username=sa
spring.datasource.password=password

# JPA Configuration
spring.jpa.database-platform=org.hibernate.dialect.H2Dialect
spring.jpa.hibernate.ddl-auto=create-drop
spring.jpa.show-sql=true

# H2 Console
spring.h2.console.enabled=true
spring.h2.console.path=/h2-console
```

## Chạy Ứng Dụng

### 1. Chạy từ IDE

Chạy class `DemoApplication.java` với main method.

### 2. Chạy từ Command Line

```bash
mvn spring-boot:run
```

### 3. Build và chạy JAR

```bash
mvn clean package
java -jar target/demo-0.0.1-SNAPSHOT.jar
```

## Test API

Sau khi chạy ứng dụng, bạn có thể test các endpoint:

```bash
# Lấy tất cả users
GET http://localhost:8080/api/users

# Tạo user mới
POST http://localhost:8080/api/users
Content-Type: application/json

{
    "name": "Nguyễn Văn A",
    "email": "nguyenvana@example.com"
}

# Lấy user theo ID
GET http://localhost:8080/api/users/1

# Cập nhật user
PUT http://localhost:8080/api/users/1
Content-Type: application/json

{
    "name": "Nguyễn Văn A Updated",
    "email": "nguyenvana@example.com"
}

# Xóa user
DELETE http://localhost:8080/api/users/1
```

## Kết Luận

Spring Boot giúp bạn phát triển ứng dụng Java một cách nhanh chóng và hiệu quả. Với những kiến thức cơ bản này, bạn có thể bắt đầu xây dựng các ứng dụng web phức tạp hơn.

Trong các bài viết tiếp theo, tôi sẽ hướng dẫn về:
- Spring Security
- Spring Data JPA nâng cao
- Testing với Spring Boot
- Docker và deployment

Hãy thử tạo ứng dụng đầu tiên của bạn và chia sẻ kết quả nhé! 🚀

