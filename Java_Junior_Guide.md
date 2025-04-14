# 🧠 Junior Java Backend Developer Guide

A comprehensive summary of core Java backend concepts every junior developer should know. Includes OOP principles, Spring ecosystem, testing, Docker, Git, SQL, and more.

---

## 📖 Table of Contents
- [OOP Principles](#-oop-principles-object-oriented-programming)
- [Java Collections](#-java-collections)
- [Important Keywords](#-important-keywords)
- [Abstract Class vs Interface](#-abstract-class-vs-interface)
- [Lambda and Streams](#-lambda-and-streams)
- [Exception Handling](#-exception-handling)
- [Spring Core Concepts](#-spring-core-concepts)
- [REST API](#-rest-api)
- [MVC in Spring](#-mvc-in-spring)
- [Spring Data JPA](#-spring-data-jpa)
- [Unit Testing with JUnit](#-unit-testing-with-junit)
- [Mockito](#-mockito)
- [Spring Security + JWT](#-spring-security--jwt)
- [Git Basics](#-git-basics)
- [Docker](#-docker)
- [Spring Events](#-spring-events)
- [CORS Configuration](#-cors-configuration)
- [Password Encoding](#-password-encoding)
- [SQL and DB Practice](#-sql-and-db-practice)

---

## 🧠 OOP Principles (Object-Oriented Programming)

### Encapsulation
Encapsulation hides an object's internal state and exposes behavior through methods. This protects data from unauthorized access.

```java
public class Person {
    private int age;
    public int getAge() { return age; }
    public void setAge(int age) { if (age > 0) this.age = age; }
}
```

### Inheritance
Inheritance allows a class to use fields and methods of another class. It enables code reuse and creates a class hierarchy.

```java
class Car extends Vehicle {}
```

### Abstraction
Abstraction hides internal logic and shows only necessary details. It simplifies complexity using abstract classes or interfaces.

```java
abstract class Animal { abstract void sound(); }
```

### Polymorphism
Polymorphism allows objects to behave differently through the same interface. It increases flexibility and reusability.

```java
Vehicle v = new Car();
v.move();
```

---

## 📚 Java Collections

### List
An ordered collection that allows duplicates.
```java
List<String> names = new ArrayList<>();
```

### Set
A collection that doesn’t allow duplicates. HashSet doesn’t preserve order.
```java
Set<String> ids = new HashSet<>();
```

### LinkedList
Efficient for insertions and deletions, slower for access by index.
```java
LinkedList<String> list = new LinkedList<>();
```

### Map
Stores key-value pairs. Keys must be unique.
```java
Map<String, Integer> ageMap = new HashMap<>();
```

---

## 🧩 Important Keywords

### static
Belongs to the class, not to instances. Shared across all objects.
```java
static int count;
```

### final
Marks a variable as constant, a method as unoverridable, or a class as unextendable.
```java
final int a = 5;
```

### abstract
Defines a method or class that must be implemented by a subclass.
```java
abstract class Shape { abstract void draw(); }
```

---

## 🧷 Abstract Class vs Interface
- **Abstract class** can have state and both abstract and concrete methods.
- **Interface** is a contract with public abstract methods, can be implemented by multiple classes.

---

## 🧪 Lambda and Streams
Lambdas simplify code by removing boilerplate. Streams provide data processing operations.

```java
list.stream().map(x -> x + 1).filter(x -> x > 5).toList();
```

### Common operators:
- `map()` – transforms
- `filter()` – filters
- `reduce()` – reduces to one result
- `collect()` – collects into a list

### Optional
Avoids `null` checks.
```java
Optional<String> name = Optional.of("Alice");
```

---

## ⚠️ Exception Handling
Exceptions signal runtime issues. Java has checked and unchecked exceptions.

### Checked Exceptions
Must be declared or handled (e.g., IOException).
```java
try {
    BufferedReader reader = new BufferedReader(new FileReader("file.txt"));
} catch (IOException e) {
    e.printStackTrace();
}
```

### Unchecked Exceptions
Runtime exceptions that don’t require declaration (e.g., NullPointerException).
```java
String s = null;
s.length(); // NullPointerException
```

### try-with-resources
Automatically closes resources.
```java
try (Scanner scanner = new Scanner(System.in)) {
    System.out.println(scanner.nextLine());
}
```

---

## 🌐 REST API
REST is a stateless architectural style for building APIs.

### HTTP Methods
- `GET`: retrieve resource
- `POST`: create resource
- `PUT`: update/replace resource
- `PATCH`: partial update
- `DELETE`: delete resource

```java
@GetMapping("/users")
public List<User> getAll() { ... }
```

---

## 🧭 MVC in Spring
- **Model** – data and logic
- **View** – UI layer
- **Controller** – handles input and connects model to view

```java
@RestController
@RequestMapping("/api")
class MyController {}
```

---

## 🗂️ Spring Data JPA
Simplifies DB access using repositories.

### Entity Example
```java
@Entity
class User {
    @Id @GeneratedValue
    Long id;
    String name;
    @OneToMany(mappedBy = "user", cascade = CascadeType.ALL)
    List<Order> orders;
}

@Entity
class Order {
    @Id @GeneratedValue
    Long id;
    @ManyToOne
    User user;
}
```

### Repository
```java
@Repository
interface UserRepo extends JpaRepository<User, Long> {
    List<User> findByName(String name);
}
```

### Custom Query
```java
@Query("SELECT u FROM User u WHERE u.age > :age")
List<User> findOlderThan(@Param("age") int age);
```

---

## 🧪 Unit Testing with JUnit
Write small tests for individual units.
```java
@Test
void testAdd() {
    assertEquals(4, calculator.add(2, 2));
}
```

### Setup methods
```java
@BeforeEach void init() { ... }
@AfterEach void cleanup() { ... }
```

---

## 🧪 Mockito
Mock dependencies and control behavior.
```java
@Mock UserRepository userRepo;
@InjectMocks UserService userService;

@Test
void testFindUser() {
    when(userRepo.findById(1L)).thenReturn(Optional.of(new User()));
    assertNotNull(userService.getUserById(1L));
}
```

---

## 🔐 Spring Security + JWT
Secure APIs with tokens.

### JWT Auth Flow
- Client logs in → gets token
- Sends token in `Authorization: Bearer <token>`

### Config Example
```java
@Bean
SecurityFilterChain security(HttpSecurity http) throws Exception {
    return http.csrf().disable()
        .authorizeHttpRequests(auth -> auth
            .requestMatchers("/api/auth/**").permitAll()
            .anyRequest().authenticated())
        .sessionManagement(session -> session.sessionCreationPolicy(SessionCreationPolicy.STATELESS))
        .addFilterBefore(jwtFilter, UsernamePasswordAuthenticationFilter.class)
        .build();
}
```

---

## 💬 Git Basics
Version control system for tracking code changes.
```bash
git pull
git branch
git merge
git rebase
git checkout -b feature/login
```

### Commit Messages
- `feat: add login endpoint`
- `fix: handle null pointer`

---

## 📦 Docker
Containerization platform to package apps.

### Dockerfile
```Dockerfile
FROM openjdk:17-jdk-slim
COPY target/app.jar app.jar
ENTRYPOINT ["java", "-jar", "app.jar"]
```

### docker-compose.yml
```yaml
version: '3.8'
services:
  app:
    build: .
    ports:
      - "8080:8080"
  db:
    image: mysql:8
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: app_db
```

---

## 🪝 Spring Events
Useful for decoupling logic. Events can be triggered and handled asynchronously.
```java
@Component
class OrderListener {
    @EventListener
    public void handleOrderCreated(OrderCreatedEvent event) {
        System.out.println("Order created: " + event.getOrderId());
    }
}
```

---

## 🌍 CORS Configuration
Allow cross-origin requests (needed when frontend is separate).
```java
@Bean
CorsConfigurationSource corsConfigurationSource() {
    CorsConfiguration config = new CorsConfiguration();
    config.addAllowedOrigin("*");
    config.addAllowedMethod("*");
    config.addAllowedHeader("*");
    UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
    source.registerCorsConfiguration("/**", config);
    return source;
}
```

---

## 🔐 Password Encoding
Never store plain passwords. Use hashing.
```java
@Bean
public PasswordEncoder passwordEncoder() {
    return new BCryptPasswordEncoder();
}
```

---

## 🗃️ SQL and DB Practice

### SQL Concepts
- `JOIN`, `GROUP BY`, `HAVING`, subqueries
- Indexes for performance
- Foreign keys to link tables
- ACID: Atomicity, Consistency, Isolation, Durability

### ORM Relationships
```java
@OneToMany(mappedBy = "user", cascade = CascadeType.ALL)
private List<Order> orders;

@ManyToOne
private User user;
```
