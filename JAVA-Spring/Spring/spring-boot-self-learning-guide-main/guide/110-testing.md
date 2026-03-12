# Testing Spring Boot Applications

## Table of Contents
- [Introduction](#introduction)
- [Spring Boot Testing Support](#spring-boot-testing-support)
- [Unit Testing](#unit-testing)
- [Slice Testing](#slice-testing)
- [Integration Testing](#integration-testing)
- [Testing Secured REST API Endpoints](#testing-secured-rest-api-endpoints)
- [Best Practices](#best-practices)
- [Assignment](#assignment)

## Introduction

Testing is crucial for ensuring the reliability and maintainability of Spring Boot applications. 
Spring Boot provides comprehensive testing support through various testing annotations and utilities 
that make it easy to write unit tests, slice tests, and integration tests.

## Spring Boot Testing Support

Spring Boot provides a wide range of tools and technologies for testing Spring Boot applications.

The `spring-boot-starter-test` transitively includes several commonly used testing libraries:

- **JUnit 5/6**: The standard testing framework for Java applications
- **Spring Test**: Spring Framework's testing support
- **AssertJ**: Fluent assertion library
- **Hamcrest**: Matcher library for assertions
- **Mockito**: Mocking framework
- **JSONassert**: JSON assertion library
- **JsonPath**: XPath for JSON

### Adding Spring Boot Test Dependencies

Add the Spring Boot Test starter to your `pom.xml`:

```xml
<!--  Upto Spring Boot 3.x  -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>

<!--  From Spring Boot 4.x, we need to add test starters for specific technologies  -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-webmvc-test</artifactId>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa-test</artifactId>
    <scope>test</scope>
</dependency>
```

## Unit Testing

Unit tests focus on testing individual components in isolation without loading the Spring context.

### Testing Service Layer without Spring
While writing unit tests for service layer components, we should be able to instantiate the component 
and invoke its methods without loading the Spring context.
If the subject under test depends on other components, we can use Mockito mocks as dependencies with stubbed behaviors.

```java
@ExtendWith(MockitoExtension.class)
class OrderServiceTest {

    @Mock
    private OrderRepository orderRepository;

    @Mock
    private PaymentService paymentService;

    @Mock
    private NotificationService notificationService;

    @InjectMocks
    private OrderService orderService;

    @Test
    void shouldProcessOrderSuccessfully() {
        // Given
        Order order = new Order(1L, "ORDER-001", 100.0);
        when(orderRepository.save(any(Order.class))).thenReturn(order);
        when(paymentService.processPayment(anyDouble())).thenReturn(true);
        doNothing().when(notificationService).sendConfirmation(any(Order.class));

        // When
        Order result = orderService.placeOrder(order);

        // Then
        assertThat(result).isNotNull();
        assertThat(result.getStatus()).isEqualTo(OrderStatus.CONFIRMED);
    }

    @Test
    void shouldHandlePaymentFailure() {
        // Given
        Order order = new Order(1L, "ORDER-002", 200.0);
        when(paymentService.processPayment(anyDouble())).thenReturn(false);

        // When
        assertThatThrownBy(() -> orderService.placeOrder(order))
            .isInstanceOf(PaymentFailedException.class);

        // Then
        verify(paymentService).processPayment(200.0);
        verify(notificationService, never()).sendConfirmation(any());
    }
}
```

## Slice Testing

Slice tests focus on testing a specific "slice" of your application with minimal context loading.

### Testing Web Layer with @WebMvcTest

The `@WebMvcTest` based slice tests loads only the web layer components (controllers, filters, etc.) without the full application context.
So, if the controller depends on other components, we need to register them as mock beans using `@MockitoBean`.

```java
@WebMvcTest(BookController.class)
class BookControllerTest {

    @Autowired
    private MockMvcTester mockMvcTester;

    @MockitoBean
    private BookService bookService;

    @Test
    void shouldReturnBookWhenGetById() throws Exception {
        // Given
        Book book = new Book(1L, "Spring Boot in Action", "Craig Walls", "1234567890");
        when(bookService.getBookById(1L)).thenReturn(book);

        MvcTestResult testResult = mockMvcTester
                .post()
                .uri("/api/books/{id}", 1L)
                .contentType(MediaType.APPLICATION_JSON)
                .content(requestBody)
                .exchange();
        
        assertThat(testResult)
                .hasStatusOk()
                .bodyJson()
                .isLenientlyEqualTo("""
                          {
                             "id": 1,
                             "title": "Spring Boot in Action",
                             "author": "Craig Walls"
                          }
                        """);
    }

    @Test
    void shouldReturnNotFoundWhenBookDoesNotExist() throws Exception {
        // Given
        when(bookService.getBookById(999L))
            .thenThrow(new BookNotFoundException("Book not found with id: 999"));

        // When & Then
        mockMvcTester
                .get()
                .uri("/api/books/{id}", 999L)
                .contentType(MediaType.APPLICATION_JSON)
                .exchange()
                .assertThat()
                .hasStatus(HttpStatus.NOT_FOUND);
    }
}
```

### Testing Data Layer with @DataJpaTest

`@DataJpaTest` configures an in-memory database if you have any in-memory database driver and loads only JPA-related components.

```java
@DataJpaTest
class BookRepositoryTest {

    @Autowired
    private TestEntityManager entityManager;

    @Autowired
    private BookRepository bookRepository;

    @Test
    void shouldSaveAndFindBook() {
        // Given
        Book book = new Book(null, "Spring Boot Guide", "John Doe", "1234567890");
        entityManager.persist(book);
        entityManager.flush();

        // When
        Optional<Book> found = bookRepository.findById(book.getId());

        // Then
        assertThat(found).isPresent();
        assertThat(found.get().getTitle()).isEqualTo("Spring Boot Guide");
    }

    @Test
    void shouldFindBooksByAuthor() {
        // Given
        Book book1 = new Book(null, "Book 1", "Jane Smith", "111");
        Book book2 = new Book(null, "Book 2", "Jane Smith", "222");
        Book book3 = new Book(null, "Book 3", "John Doe", "333");

        entityManager.persist(book1);
        entityManager.persist(book2);
        entityManager.persist(book3);
        entityManager.flush();

        // When
        List<Book> books = bookRepository.findByAuthor("Jane Smith");

        // Then
        assertThat(books).hasSize(2);
        assertThat(books).extracting(Book::getAuthor)
            .containsOnly("Jane Smith");
    }
}
```

There many other slice testing annotations available such as `@JsonTest`, `@DataMongoTest`, `@JdbcTest`, etc. 

## Integration Testing

Integration tests load the complete application context and test the application as a whole.

### Using @SpringBootTest

```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@AutoConfigureMockMvc
@AutoConfigureRestTestClient
class BookIntegrationTest {

    @Autowired
    protected MockMvcTester mockMvcTester;

    @Autowired
    protected RestTestClient restTestClient;
    
    // NOTE: You can use mockMvcTester to invoke API endpoints similar to the @WebMvcTest example.
    // However, I prefer to use restTestClient for invoking API endpoints.

    @Autowired
    private BookRepository bookRepository;

    @Test
    void shouldCreateAndRetrieveBook() {
        // Given
        BookDTO newBook = new BookDTO("Integration Test Book", "Test Author", "INT-123");

        // When - Create book
        BookDTO bookDTO = restTestClient
                .post()
                .uri("/api/books")
                .contentType(MediaType.APPLICATION_JSON)
                .body(newBook)
                .exchange()
                .expectStatus().isCreated()
                .returnResult(BookDTO.class)
                .getResponseBody();
        
        // Then - Verify creation
        assertThat(bookDTO).isNotNull();
        Long bookId = bookDTO.getId();

        // When - Retrieve book
        BookDTO savedBook = restTestClient
                .get()
                .uri("/api/books/{id}", bookId)
                .exchange()
                .expectStatus().isOk()
                .returnResult(BookDTO.class)
                .getResponseBody();

        // Then - Verify retrieval
        assertThat(savedBook).isNotNull();
        assertThat(savedBook.getTitle()).isEqualTo("Integration Test Book");
    }
}
```

### Using Testcontainers for Database Testing
[Testcontainers](https://testcontainers.com/) help us to spin up application dependencies (such as databases, message brokers, and more) 
as docker containers in a controlled environment during tests. 
This is particularly useful for integration testing where the application needs to interact with external services.

Add Testcontainers dependency:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-testcontainers</artifactId>
    <scope>test</scope>
</dependency>

<!-- Upto Spring Boot 3.x (Testcontainers 1.x) -->
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

<!-- From Spring Boot 4.x (Testcontainers 2.x) -->
<dependency>
    <groupId>org.testcontainers</groupId>
    <artifactId>testcontainers-junit-jupiter</artifactId>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.testcontainers</groupId>
    <artifactId>testcontainers-postgresql</artifactId>
    <scope>test</scope>
</dependency>
```

Create a test configuration:

```java
@TestConfiguration(proxyBeanMethods = false)
public class TestContainersConfiguration {

    @Bean
    @ServiceConnection
    PostgreSQLContainer postgresContainer() {
        return new PostgreSQLContainer("postgres:18-alpine");
    }
}
```

Use Testcontainers in integration tests:

```java
import org.springframework.context.annotation.Import;

@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@Import(TestContainersConfiguration.class)
class BookIntegrationWithTestcontainersTest {

    @Autowired
    private TestRestTemplate restTemplate;

    @Autowired
    private BookRepository bookRepository;

    @Test
    void shouldPersistBookInPostgres() {
        //...
    }

    @Test
    void shouldHandleTransactions() {
        //...
    }
}
```

Now when you run the tests, Testcontainers will spin up a PostgreSQL database for you and configure the application to use it.

**NOTE:** In `@SpringBootTest` integration tests also you can use `@MockitoBean` to mock a Spring bean.

## Testing Secured REST API Endpoints

Add security test dependency:

```xml
<!-- Upto Spring Boot 3.x (Testcontainers 1.x) -->
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-test</artifactId>
    <scope>test</scope>
</dependency>

<!-- From Spring Boot 4.x (Testcontainers 2.x) -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security-test</artifactId>
    <scope>test</scope>
</dependency>
```

### Testing with Spring Security

You can use `@WithMockUser`, `@WithUserDetails` annotations to simulate authenticated user.

You can also simulate authenticated user request using `MockMvcTester.with(user(...))` as well.

```java
@SpringBootTest
@AutoConfigureMockMvc
class SecuredBookControllerTest {

    @Autowired
    private MockMvcTester mockMvcTester;

    @Test
    void shouldDenyAccessWithoutAuthentication() throws Exception {
        mockMvcTester
                .get()
                .uri("/api/books")
                .exchange()
                .assertThat()
                .hasStatus(HttpStatus.UNAUTHORIZED);
    }

    @Test
    @WithMockUser(username = "user", roles = {"USER"})
    void shouldAllowAccessWithAuthentication() throws Exception {
        mockMvcTester
                .get()
                .uri("/api/books")
                .exchange()
                .assertThat()
                .hasStatus(HttpStatus.OK);
    }

    @Test
    void shouldAuthenticateUsingWithUser() throws Exception {
        mockMvcTester
                .get()
                .uri("/api/books")
                .with(user("user")) // <--- note this
                .exchange()
                .assertThat()
                .hasStatus(HttpStatus.OK);
    }

    @Test
    @WithUserDetails("admin@example.com")
    void shouldUseActualUserFromDatabase() throws Exception {
        mockMvcTester
                .get()
                .uri("/api/books")
                .exchange()
                .assertThat()
                .hasStatus(HttpStatus.OK);
    }
}
```

### Testing with a Real JWT Token

Instead of mocking authentication, you can generate a real JWT token using `JwtTokenProvider` and use it in your tests.

```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@AutoConfigureRestTestClient
class JwtSecuredEndpointTest {

    @Autowired
    private MockMvcTester mockMvcTester;
    
    @Autowired
    private RestTestClient restTestClient;
    
    @Autowired
    private JwtTokenProvider tokenProvider;

    @Test
    void shouldAccessProtectedEndpointWithValidToken() {
        // Given
        String token = tokenProvider.generateToken("user@example.com", List.of("ROLE_USER"));

        // When
        
        //Using MockMvcTester
        assertThat(mockMvcTester.get().uri("/api/books")
                .header("Authorization", "Bearer " + token))
                .hasStatusOk();
        
        // Using RestTestClient        
        BookDTO[] books = restTestClient
                .get()
                .uri("/api/books")
                .header("Authorization", "Bearer " + token)
                .exchange()
                .expectStatus()
                .isOk()
                .returnResult(BookDTO[].class)
                .getResponseBody();
        
        // Then
        assertThat(books).isNotNull();
    }
}
```

## Best Practices

1. **Use the Right Test Type**:
   - Unit tests for business logic
   - Slice tests for specific layers
   - Integration tests for end-to-end flows

2. **Follow AAA Pattern**: Arrange, Act, Assert
   ```java
   @Test
   void testMethod() {
       // Arrange (Given)
       // Act (When)
       // Assert (Then)
   }
   ```

3. **Use Descriptive Test Names**:
   ```java
   shouldReturnBookWhenValidIdProvided()
   shouldThrowExceptionWhenBookNotFound()
   ```

4. **Isolate Tests**: Each test should be independent
   ```java
   @BeforeEach
   void setUp() {
       bookRepository.deleteAll();
   }
   ```

5. **Use Test Fixtures**: Create reusable test data
   ```java
   class BookFixtures {
       static Book createDefaultBook() {
           return new Book(null, "Default Book", "Default Author", "123");
       }
   }
   ```

6. **External Dependencies**: Use Testcontainers or WireMock for external services

7. **Test Edge Cases**: Include boundary conditions, null values, and error scenarios

8. **Use AssertJ for Readable Assertions**:
   ```java
   assertThat(book)
       .isNotNull()
       .extracting(Book::getTitle)
       .isEqualTo("Expected Title");
   ```

## Assignment

Create a comprehensive test suite for a jblogger with the following requirements:

1. **Unit Tests**:
   - Test `PostService` with mocked `PostRepository`
   - Test `UserService` with mocked dependencies
   - Include edge cases and error scenarios

2. **Slice Tests**:
   - `@WebMvcTest` for `PostController`
   - `@DataJpaTest` for `PostRepository` custom queries
   - Test validation constraints

3. **Integration Tests**:
   - Use `@SpringBootTest` with Testcontainers (PostgreSQL)
   - Test all API endpoints using `TestRestTemplate` or `MockMvcTester` 

4. **Security Tests**:
   - Test authenticated and unauthenticated access
   - Test role-based authorization (USER, ADMIN)
   - Test JWT token validation

5. **Coverage Goals**:
   - Aim for >80% code coverage
   - Include happy path and error scenarios

---
[Home](../README.md) | [← Previous: Spring Security JWT](100-spring-security-jwt.md) | [Next: Docker Compose Deployment →](120-docker-compose-deployment.md)
