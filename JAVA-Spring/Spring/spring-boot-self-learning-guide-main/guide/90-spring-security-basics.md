# Spring Security Basics

## Table of Contents
- [Web Application Security Overview](#web-application-security-overview)
- [Java Web Application Security](#java-web-application-security)
- [Spring Security Architecture](#spring-security-architecture)
- [Spring Security Key Components](#spring-security-key-components)
- [Spring Security in Web Applications](#spring-security-in-web-applications)
- [Method-Level Security](#method-level-security)
- [Assignments](#assignments)

## Web Application Security Overview

Security is a critical aspect of web applications. You need to protect your resources from unauthorized access 
and ensure that only authenticated users can access private resources.

### Public vs Private Resources

In most web applications, you have two types of resources:

**1. Public Resources** - Available to everyone without authentication
- Homepage
- About page
- Contact page
- Product listing pages
- Login page
- Registration page

**2. Private Resources** - Available only to authenticated users
- User profile
- Dashboard
- Shopping cart
- Order history
- Admin panel

**Real-World Example:**
Think of a shopping website like Amazon:
- Anyone can browse products (public)
- Only logged-in users can add to cart or make purchases (private)
- Only admins can manage products and orders (private + role-based)

```
┌─────────────────────────────────────────┐
│         Web Application                 │
├─────────────────────────────────────────┤
│  Public Resources                       │
│  ✓ /home        (Everyone)              │
│  ✓ /products    (Everyone)              │
│  ✓ /login       (Everyone)              │
├─────────────────────────────────────────┤
│  Private Resources                      │
│  ✓ /profile     (Authenticated Users)   │
│  ✓ /cart        (Authenticated Users)   │
│  ✓ /admin       (Admin Role Only)       │
└─────────────────────────────────────────┘
```

### Role-Based Access Control (RBAC)

RBAC is a method to control access to resources based on the user's role in the organization.

**Common Roles:**
- **USER** - Regular users with basic access
- **ADMIN** - Administrators with full access
- **MANAGER** - Managers with specific permissions
- **GUEST** - Limited access

**Example Scenario:**

```
User: John (Role: USER)
- Can view products ✓
- Can place orders ✓
- Can view own profile ✓
- Cannot access admin panel ✗

User: Sarah (Role: ADMIN)
- Can view products ✓
- Can place orders ✓
- Can view own profile ✓
- Can access admin panel ✓
- Can manage all users ✓
```

**RBAC Benefits:**
1. **Simplified Management** - Assign roles instead of individual permissions
2. **Scalability** - Easy to add new users with predefined roles
3. **Security** - Principle of least privilege (users get only what they need)
4. **Compliance** - Meet regulatory requirements

## Java Web Application Security

Before diving into Spring Security, let's understand how security works in traditional Java web applications.

### Servlets to Handle HTTP Requests

Servlets are Java classes that handle HTTP requests and responses.

**Example Servlet:**
```java
@WebServlet("/profile")
public class ProfileServlet extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {

        // Check if user is logged in
        HttpSession session = request.getSession(false);
        if (session == null || session.getAttribute("user") == null) {
            response.sendRedirect("/login");
            return;
        }

        // User is authenticated, show profile
        response.getWriter().println("Welcome to your profile!");
    }
}
```

**Problems with Manual Security Checks:**
- Code duplication in every servlet
- Easy to forget security checks
- Hard to maintain
- No centralized security logic

### Filters to Intercept Requests

Filters are Java components that intercept requests before they reach servlets. They're perfect for implementing security checks.

**Filter Lifecycle:**
```
Browser → Filter → Servlet → Response
           ↓
    Security Check
```

**Example Security Filter:**
```java
@WebFilter("/secure/*")
public class AuthenticationFilter implements Filter {

    @Override
    public void doFilter(ServletRequest request, ServletResponse response,
                         FilterChain chain) throws IOException, ServletException {

        HttpServletRequest httpRequest = (HttpServletRequest) request;
        HttpServletResponse httpResponse = (HttpServletResponse) response;

        // Check if user is authenticated
        HttpSession session = httpRequest.getSession(false);
        if (session == null || session.getAttribute("user") == null) {
            // Not authenticated - redirect to login
            httpResponse.sendRedirect("/login");
            return;
        }

        // Authenticated - continue to the requested resource
        chain.doFilter(request, response);
    }
}
```

**How Filters Work:**

1. **Request arrives** at `/secure/profile`
2. **Filter intercepts** the request
3. **Filter checks** if user is authenticated
4. **If authenticated**: Pass request to servlet
5. **If not authenticated**: Redirect to login page

**Filter Pattern Matching:**
```java
@WebFilter("/admin/*")    // Protects all admin URLs
@WebFilter("/secure/*")   // Protects all secure URLs
@WebFilter("/*")          // Intercepts ALL requests
```

### Authentication and Authorization

**Authentication** - "Who are you?"
- Verifies the identity of the user
- Checks username and password
- Confirms user is who they claim to be

**Authorization** - "What can you do?"
- Determines what resources the user can access
- Checks user's roles and permissions
- Enforces access control rules

**Example Flow:**

```
┌──────────────────────────────────────────────────────┐
│ 1. User Login Request                                │
│    Username: john@example.com                        │
│    Password: secret123                               │
└────────────────┬─────────────────────────────────────┘
                 ↓
┌──────────────────────────────────────────────────────┐
│ 2. Authentication (Who are you?)                     │
│    - Verify username exists                          │
│    - Check password matches                          │
│    - Load user roles                                 │
│    Result: ✓ Authenticated (Roles: USER, MANAGER)    │
└────────────────┬─────────────────────────────────────┘
                 ↓
┌──────────────────────────────────────────────────────┐
│ 3. User tries to access /admin/users                 │
└────────────────┬─────────────────────────────────────┘
                 ↓
┌──────────────────────────────────────────────────────┐
│ 4. Authorization (What can you do?)                  │
│    Required Role: ADMIN                              │
│    User Roles: USER, MANAGER                         │
│    Result: ✗ Access Denied (Missing ADMIN role)      │
└──────────────────────────────────────────────────────┘
```

**Traditional Java Implementation:**
```java
public class SecurityHelper {

    // Authentication
    public User authenticate(String username, String password) {
        User user = userRepository.findByUsername(username);
        if (user == null) {
            throw new AuthenticationException("User not found");
        }

        if (!passwordMatches(password, user.getPassword())) {
            throw new AuthenticationException("Invalid password");
        }

        return user;
    }

    // Authorization
    public boolean authorize(User user, String requiredRole) {
        return user.getRoles().contains(requiredRole);
    }
}
```

**Limitations of Manual Implementation:**
- Complex code to maintain
- Security vulnerabilities if not done correctly
- Need to handle sessions, cookies, CSRF tokens manually
- Password encryption and hashing
- Remember-me functionality
- Account lockout after failed attempts

**This is where Spring Security comes in!** It provides all these features out of the box.

## Spring Security Architecture

Spring Security is a powerful and highly customizable authentication and access-control framework. 
It's the de-facto standard for securing Spring-based applications.

### Spring Security Filter Chain

Spring Security uses a chain of filters to handle security concerns. 
When a request comes in, it passes through multiple security filters before reaching your controller.

**Filter Chain Flow:**

```
HTTP Request
     ↓
┌────────────────────────────────────────────┐
│  Spring Security Filter Chain              │
├────────────────────────────────────────────┤
│  1. SecurityContextPersistenceFilter       │
│     (Load security context from session)   │
├────────────────────────────────────────────┤
│  2. LogoutFilter                           │
│     (Handle logout requests)               │
├────────────────────────────────────────────┤
│  3. UsernamePasswordAuthenticationFilter   │
│     (Process login form submission)        │
├────────────────────────────────────────────┤
│  4. BasicAuthenticationFilter              │
│     (Handle HTTP Basic authentication)     │
├────────────────────────────────────────────┤
│  5. ExceptionTranslationFilter             │
│     (Handle security exceptions)           │
├────────────────────────────────────────────┤
│  6. FilterSecurityInterceptor              │
│     (Check authorization/access control)   │
└──────────────────┬─────────────────────────┘
                   ↓
           Your Controller
```

**Key Filters Explained:**

1. **SecurityContextPersistenceFilter**
   - Loads the security context (user info) from session
   - Makes it available throughout the request

2. **UsernamePasswordAuthenticationFilter**
   - Processes login form submissions
   - Authenticates username and password
   - Creates authentication token

3. **ExceptionTranslationFilter**
   - Catches security exceptions
   - Redirects to login page if not authenticated
   - Shows access denied page if not authorized

4. **FilterSecurityInterceptor**
   - Final filter in the chain
   - Checks if user has required roles/permissions
   - Either allows access or throws AccessDeniedException

**Example Request Flow:**

```
User submits login form
  ↓
UsernamePasswordAuthenticationFilter intercepts
  ↓
Extracts username="john" password="secret123"
  ↓
Calls AuthenticationManager.authenticate()
  ↓
UserDetailsService loads user from database
  ↓
PasswordEncoder compares passwords
  ↓
If successful: Create Authentication object
  ↓
Store in SecurityContext
  ↓
Redirect to homepage
```

### Spring Security Configuration

Spring Security can be configured in multiple ways:

**1. Default Configuration (Zero Configuration)**
- Spring Boot auto-configures security
- All endpoints protected by default
- Single user with generated password

**2. Properties-based Configuration**
- Configure username/password in application.properties
- Simple but limited customization

**3. Java-based Configuration**
- Create a configuration class
- Full control over security behavior
- Most flexible approach

**Configuration Class Structure:**

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/public/**").permitAll()
                .requestMatchers("/admin/**").hasRole("ADMIN")
                .anyRequest().authenticated()
            )
            .formLogin(form -> form
                .loginPage("/login")
                .permitAll()
            )
            .logout(logout -> logout
                .logoutSuccessUrl("/")
            );

        return http.build();
    }
}
```

### Spring Security Authentication Methods

Spring Security supports multiple authentication methods:

#### 1. Basic Authentication

Sends credentials with each request in HTTP headers (Base64 encoded).

```
GET /api/users HTTP/1.1
Authorization: Basic am9objpzZWNyZXQxMjM=
```

**Configuration:**
```java
http.httpBasic(Customizer.withDefaults());
```

**Use Case:** REST APIs, simple integrations

**Pros:**
- Simple to implement
- Stateless (no session)

**Cons:**
- Credentials sent with every request
- Must use HTTPS
- No logout mechanism

#### 2. Form-Based Login

Traditional HTML form for login.

```html
<form action="/login" method="POST">
    <input type="text" name="username" />
    <input type="password" name="password" />
    <button type="submit">Login</button>
</form>
```

**Configuration:**
```java
http.formLogin(form -> form
    .loginPage("/login")
    .defaultSuccessUrl("/home")
    .permitAll()
);
```

**Use Case:** Web applications with UI

**Pros:**
- User-friendly
- Session-based
- Supports remember-me

**Cons:**
- Requires session management
- CSRF protection needed

#### 3. JWT (JSON Web Token)

Token-based authentication for stateless applications.

```
GET /api/users HTTP/1.1
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

**Use Case:** Modern SPAs, microservices

**Pros:**
- Stateless
- Scalable
- Works across domains

**Cons:**
- Token revocation complexity
- Larger payload size

#### 4. OAuth2

Delegated authentication using third-party providers.

**Configuration:**
```java
http.oauth2Login(oauth -> oauth
    .loginPage("/login")
    .defaultSuccessUrl("/home")
);
```

**Use Case:** Social login (Google, Facebook, GitHub)

**Pros:**
- No password management
- Trusted providers
- Better user experience

**Cons:**
- Depends on external service
- Complex setup

---

## Spring Security Key Components

Understanding these core components is essential for working with Spring Security.

### 1. UserDetails

`UserDetails` is an interface that represents a user in Spring Security. It provides core user information.

**Interface Definition:**
```java
public interface UserDetails {
    String getUsername();
    String getPassword();
    Collection<? extends GrantedAuthority> getAuthorities();
    boolean isAccountNonExpired();
    boolean isAccountNonLocked();
    boolean isCredentialsNonExpired();
    boolean isEnabled();
}
```

**Implementation Example:**
```java
public class CustomUserDetails implements UserDetails {

    private String username;
    private String password;
    private List<GrantedAuthority> authorities;
    private boolean enabled;

    public CustomUserDetails(User user) {
        this.username = user.getEmail();
        this.password = user.getPassword();
        this.authorities = user.getRoles().stream()
            .map(role -> new SimpleGrantedAuthority("ROLE_" + role))
            .collect(Collectors.toList());
        this.enabled = user.isActive();
    }

    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        return authorities;
    }

    @Override
    public String getPassword() {
        return password;
    }

    @Override
    public String getUsername() {
        return username;
    }

    @Override
    public boolean isAccountNonExpired() {
        return true;
    }

    @Override
    public boolean isAccountNonLocked() {
        return true;
    }

    @Override
    public boolean isCredentialsNonExpired() {
        return true;
    }

    @Override
    public boolean isEnabled() {
        return enabled;
    }
}
```

**Spring's Default Implementation:**
```java
// Quick way to create UserDetails
UserDetails user = org.springframework.security.core.userdetails.User
    .withUsername("john")
    .password("{bcrypt}$2a$10$...")
    .roles("USER", "ADMIN")
    .build();
```

**Key Points:**
- Represents the authenticated user
- Contains credentials and authorities
- Can be customized to include additional fields (email, phone, etc.)

### 2. UserDetailsService

`UserDetailsService` is responsible for loading user-specific data. It has one method: `loadUserByUsername()`.

**Interface Definition:**
```java
public interface UserDetailsService {
    UserDetails loadUserByUsername(String username) throws UsernameNotFoundException;
}
```

**Implementation Example:**
```java
@Service
public class CustomUserDetailsService implements UserDetailsService {

    @Autowired
    private UserRepository userRepository;

    @Override
    public UserDetails loadUserByUsername(String username)
            throws UsernameNotFoundException {

        // Load user from database
        User user = userRepository.findByEmail(username)
            .orElseThrow(() -> new UsernameNotFoundException(
                "User not found: " + username));

        // Convert to UserDetails
        return new CustomUserDetails(user);
    }
}
```

**Real-World Example with JPA:**
```java
// Entity
@Entity
@Table(name = "users")
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(unique = true, nullable = false)
    private String email;

    @Column(nullable = false)
    private String password;

    @ElementCollection(fetch = FetchType.EAGER)
    private Set<String> roles = new HashSet<>();

    private boolean active = true;

    // Getters and setters
}

// Repository
public interface UserRepository extends JpaRepository<User, Long> {
    Optional<User> findByEmail(String email);
}

// UserDetailsService
@Service
public class DatabaseUserDetailsService implements UserDetailsService {

    @Autowired
    private UserRepository userRepository;

    @Override
    public UserDetails loadUserByUsername(String email)
            throws UsernameNotFoundException {

        User user = userRepository.findByEmail(email)
            .orElseThrow(() -> new UsernameNotFoundException(
                "User not found with email: " + email));

        return org.springframework.security.core.userdetails.User
            .withUsername(user.getEmail())
            .password(user.getPassword())
            .roles(user.getRoles().toArray(new String[0]))
            .disabled(!user.isActive())
            .build();
    }
}
```

**Key Points:**
- Called by Spring Security during authentication
- Should throw `UsernameNotFoundException` if user not found
- Typically loads user from database, LDAP, or external API

### 3. PasswordEncoder

`PasswordEncoder` is used to encode passwords and verify them during authentication.

**Interface Definition:**
```java
public interface PasswordEncoder {
    String encode(CharSequence rawPassword);
    boolean matches(CharSequence rawPassword, String encodedPassword);
}
```

**Common Encoders:**

1. **BCryptPasswordEncoder** (Recommended)
```java
@Bean
public PasswordEncoder passwordEncoder() {
    return new BCryptPasswordEncoder();
}
```

2. **Pbkdf2PasswordEncoder**
```java
@Bean
public PasswordEncoder passwordEncoder() {
    return new Pbkdf2PasswordEncoder();
}
```

3. **SCryptPasswordEncoder**
```java
@Bean
public PasswordEncoder passwordEncoder() {
    return new SCryptPasswordEncoder();
}
```

**Usage Example:**
```java
@Service
public class UserService {
    
    private final PasswordEncoder passwordEncoder;
    private final UserRepository userRepository;
    
    //constructor

    public void registerUser(String email, String rawPassword) {
        // Encode password before saving
        String encodedPassword = passwordEncoder.encode(rawPassword);

        User user = new User();
        user.setEmail(email);
        user.setPassword(encodedPassword);
        user.getRoles().add("USER");

        userRepository.save(user);
    }

    public boolean checkPassword(String email, String rawPassword) {
        User user = userRepository.findByEmail(email)
            .orElseThrow(() -> new UsernameNotFoundException("User not found"));

        // Verify password
        return passwordEncoder.matches(rawPassword, user.getPassword());
    }
}
```

**BCrypt Example:**
```java
PasswordEncoder encoder = new BCryptPasswordEncoder();

// Encoding
String rawPassword = "myPassword123";
String encoded = encoder.encode(rawPassword);
// Output: $2a$10$xjfHKf7d8YhKJF9fJK8f.eW...

// Verification
boolean matches = encoder.matches("myPassword123", encoded);
// Output: true

boolean matches2 = encoder.matches("wrongPassword", encoded);
// Output: false
```

**Password Storage Format:**
```
{algorithm}encodedPassword

Examples:
{bcrypt}$2a$10$xjfHKf7d8YhKJF9fJK8f.eW...
{pbkdf2}$2a$10$xjfHKf7d8YhKJF9fJK8f.eW...
{noop}plainTextPassword  // No encoding (NOT RECOMMENDED)
```

**Key Points:**
- **Never store plain text passwords**
- BCrypt is recommended (adaptive, includes salt)
- Same password produces different encoded values (due to salt)
- One-way encoding (cannot decode back to original)

### 4. AuthenticationManager

`AuthenticationManager` is the main interface for authentication in Spring Security.

**Interface Definition:**
```java
public interface AuthenticationManager {
    Authentication authenticate(Authentication authentication)
        throws AuthenticationException;
}
```

**Default Implementation: ProviderManager**

`ProviderManager` delegates to a list of `AuthenticationProvider` instances.

**Authentication Flow:**

```
1. User submits credentials
         ↓
2. Create Authentication object (unauthenticated)
         ↓
3. AuthenticationManager.authenticate(auth)
         ↓
4. ProviderManager loops through AuthenticationProviders
         ↓
5. DaoAuthenticationProvider:
   - Calls UserDetailsService.loadUserByUsername()
   - Loads UserDetails from database
   - Calls PasswordEncoder.matches()
   - Compares passwords
         ↓
6. If successful: Return Authentication object (authenticated)
   If failed: Throw AuthenticationException
         ↓
7. Store Authentication in SecurityContext
```

**Configuration Example:**
```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }

    @Bean
    AuthenticationManager authenticationManager(UserDetailsService uds, PasswordEncoder pe) {
        var authProvider = new DaoAuthenticationProvider(uds);
        authProvider.setPasswordEncoder(pe);

        return new ProviderManager(authProvider);
    }
}
```

**Manual Authentication Example:**
```java
@RestController
@RequestMapping("/auth")
public class AuthController {

    @Autowired
    private AuthenticationManager authenticationManager;

    @PostMapping("/login")
    public ResponseEntity<String> login(@RequestBody LoginRequest request) {
        try {
            // Create unauthenticated token
            Authentication authentication = new UsernamePasswordAuthenticationToken(
                request.getUsername(),
                request.getPassword()
            );

            // Authenticate
            Authentication authenticated =
                authenticationManager.authenticate(authentication);

            // Store in security context
            SecurityContextHolder.getContext().setAuthentication(authenticated);

            return ResponseEntity.ok("Login successful");

        } catch (AuthenticationException e) {
            return ResponseEntity.status(HttpStatus.UNAUTHORIZED)
                .body("Invalid credentials");
        }
    }
}
```

**Key Components Working Together:**

```
┌─────────────────────────────────────────────────────┐
│         AuthenticationManager                       │
│           (ProviderManager)                         │
└────────────────┬────────────────────────────────────┘
                 ↓
┌─────────────────────────────────────────────────────┐
│      DaoAuthenticationProvider                      │
├─────────────────────────────────────────────────────┤
│  - Uses UserDetailsService                          │
│  - Uses PasswordEncoder                             │
└────────────┬────────────────────┬───────────────────┘
             ↓                    ↓
┌────────────────────┐  ┌─────────────────────┐
│ UserDetailsService │  │  PasswordEncoder    │
│                    │  │                     │
│ loadUserByUsername │  │  encode()           │
│                    │  │  matches()          │
└────────────────────┘  └─────────────────────┘
```

## Spring Security in Web Applications

Let's explore different ways to configure Spring Security in a Spring Boot application, from simplest to most advanced.

### 1. Default Spring Security Configuration

When you add Spring Security dependency, Spring Boot automatically configures security.

**Add Dependency:**
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

**What Happens Automatically:**

1. **All endpoints/urls are protected** - Requires authentication for every URL
2. **Default user created** - Username: `user`
3. **Random password generated** - Printed in console logs
4. **Default login page** - Auto-generated at `/login`
5. **HTTP Basic authentication** - Enabled by default
6. **CSRF protection** - Enabled by default

**Console Output:**
```
Using generated security password: 8e4e5c3a-6f5b-4d8e-9a3f-2c1d5e6f7a8b
```

To use Thymeleaf views, add the following dependency:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```

Create thymeleaf templates in `src/main/resources/templates` directory.

**Testing Default Security:**
```java
@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}

@RestController
public class HomeController {

    @GetMapping("/")
    public String home() {
        return "Welcome Home!";
    }

    @GetMapping("/user")
    public String user() {
        return "User Page";
    }
}
```

**Accessing the Application:**
- Navigate to `http://localhost:8080/`
- Redirected to `/login`
- Enter username: `user`
- Enter password: (from console)
- Access granted

**Pros:**
- Zero configuration required
- Quick setup for testing

**Cons:**
- Not suitable for production
- Password changes on every restart
- Single user only
- No customization

### 2. Security Configuration with Properties

Configure basic security settings using `application.properties` or `application.yml`.

**application.properties:**
```properties
# Single user configuration
spring.security.user.name=admin
spring.security.user.password=admin123
spring.security.user.roles=ADMIN
```

**Testing:**
```bash
curl -u admin:admin123 http://localhost:8080/user
```

**Limitations:**
- Only one user can be configured
- No password encryption
- Cannot configure URL-specific security
- Not suitable for production

### 3. In-Memory Authentication

Configure multiple users in memory using Java configuration.

**SecurityConfig.java:**
```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/", "/home", "/public/**").permitAll()
                .requestMatchers("/admin/**").hasRole("ADMIN")
                .requestMatchers("/user/**").hasAnyRole("USER", "ADMIN")
                .anyRequest().authenticated()
            )
            .formLogin(form -> form
                .loginPage("/login")
                .defaultSuccessUrl("/home")
                .permitAll()
            )
            .logout(logout -> logout
                .logoutSuccessUrl("/")
                .permitAll()
            );

        return http.build();
    }

    @Bean
    public UserDetailsService userDetailsService(PasswordEncoder passwordEncoder) {
        UserDetails user = User.builder()
            .username("user")
            .password(passwordEncoder.encode("user123"))
            .roles("USER")
            .build();

        UserDetails admin = User.builder()
            .username("admin")
            .password(passwordEncoder.encode("admin123"))
            .roles("ADMIN", "USER")
            .build();

        UserDetails manager = User.builder()
            .username("manager")
            .password(passwordEncoder.encode("manager123"))
            .roles("MANAGER")
            .build();

        return new InMemoryUserDetailsManager(user, admin, manager);
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}
```

**URL Access Control Explained:**

```java
.requestMatchers("/", "/home", "/public/**").permitAll()
// Everyone can access: /, /home, /public/anything

.requestMatchers(HttpMethod.GET, "/api/products").permitAll()
// Everyone can access: /api/products using only HTTP GET method

.requestMatchers("/admin/**").hasRole("ADMIN")
// Only ADMIN role: /admin/users, /admin/settings

.requestMatchers("/user/**").hasAnyRole("USER", "ADMIN")
// USER or ADMIN role: /user/profile, /user/orders

.anyRequest().authenticated()
// All other URLs require authentication
```

**Testing Different Users:**

```java
@RestController
public class TestController {

    @GetMapping("/")
    public String home() {
        return "Public Home Page";
    }

    @GetMapping("/user/profile")
    public String userProfile() {
        Authentication auth = SecurityContextHolder.getContext().getAuthentication();
        return "User Profile: " + auth.getName();
    }

    @GetMapping("/admin/dashboard")
    public String adminDashboard() {
        return "Admin Dashboard";
    }
}
```

**Access Control Results:**

| URL                 | user | admin | manager | Anonymous |
|---------------------|------|-------|---------|-----------|
| /                   | ✓    | ✓     | ✓       | ✓         |
| /user/profile       | ✓    | ✓     | ✗       | ✗         |
| /admin/dashboard    | ✗    | ✓     | ✗       | ✗         |

**Pros:**
- Multiple users with different roles
- Passwords encrypted
- URL-specific security
- Good for testing and development

**Cons:**
- Users hardcoded (restart required to change)
- Not suitable for production

### 4. JDBC Authentication

Store users in a database and authenticate against it.

**Step 1: Database Schema**

Spring Security provides default schema:

```sql
-- Users table
CREATE TABLE users (
    username VARCHAR(50) NOT NULL PRIMARY KEY,
    password VARCHAR(100) NOT NULL,
    enabled BOOLEAN NOT NULL
);

-- Authorities table
CREATE TABLE authorities (
    username VARCHAR(50) NOT NULL,
    authority VARCHAR(50) NOT NULL,
    FOREIGN KEY (username) REFERENCES users(username)
);

-- Optional: Create unique index
CREATE UNIQUE INDEX ix_auth_username ON authorities (username, authority);
```

**Step 2: Insert Sample Data**

```sql
-- Insert users (passwords are BCrypt encoded)
-- password for 'john' is 'john123'
INSERT INTO users (username, password, enabled)
VALUES ('john', '$2a$10$xjfHKf7d8YhKJF9fJK8f.eW5J3F5fJf8f.eW5J3F5fJf', true);

-- password for 'sarah' is 'sarah123'
INSERT INTO users (username, password, enabled)
VALUES ('sarah', '$2a$10$aK9fJK8f.eW5J3F5fJf8f.eW5J3F5fJf8f.eW5J3F5f', true);

-- Insert authorities
INSERT INTO authorities (username, authority) VALUES ('john', 'ROLE_USER');
INSERT INTO authorities (username, authority) VALUES ('sarah', 'ROLE_ADMIN');
INSERT INTO authorities (username, authority) VALUES ('sarah', 'ROLE_USER');
```

**Step 3: Security Configuration**

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Autowired
    private DataSource dataSource;

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/", "/home").permitAll()
                .requestMatchers("/admin/**").hasRole("ADMIN")
                .anyRequest().authenticated()
            )
            .formLogin(Customizer.withDefaults())
            .logout(Customizer.withDefaults());

        return http.build();
    }

    @Bean
    public UserDetailsService userDetailsService() {
        JdbcUserDetailsManager manager = new JdbcUserDetailsManager(dataSource);
        return manager;
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}
```

**Step 4: Custom Schema (Optional)**

If you have your own user table structure:

```sql
CREATE TABLE app_users (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    email VARCHAR(100) NOT NULL UNIQUE,
    password VARCHAR(100) NOT NULL,
    full_name VARCHAR(100),
    active BOOLEAN DEFAULT true
);

CREATE TABLE user_roles (
    user_id BIGINT NOT NULL,
    role VARCHAR(50) NOT NULL,
    FOREIGN KEY (user_id) REFERENCES app_users(id)
);
```

**Custom JDBC Configuration:**

```java
@Bean
public UserDetailsService userDetailsService() {
    JdbcUserDetailsManager manager = new JdbcUserDetailsManager(dataSource);

    // Custom query to load user
    manager.setUsersByUsernameQuery(
        "SELECT email, password, active FROM app_users WHERE email = ?"
    );

    // Custom query to load authorities
    manager.setAuthoritiesByUsernameQuery(
        "SELECT u.email, r.role " +
        "FROM app_users u " +
        "JOIN user_roles r ON u.id = r.user_id " +
        "WHERE u.email = ?"
    );

    return manager;
}
```

**Creating Users Programmatically:**

```java
@Service
public class UserInitService {

    @Autowired
    private JdbcUserDetailsManager userDetailsManager;

    @Autowired
    private PasswordEncoder passwordEncoder;

    @PostConstruct
    public void initUsers() {
        if (!userDetailsManager.userExists("john")) {
            UserDetails user = User.builder()
                .username("john")
                .password(passwordEncoder.encode("john123"))
                .roles("USER")
                .build();
            userDetailsManager.createUser(user);
        }
    }
}
```

**Pros:**
- Users stored in database
- Can add/remove users without restarting
- Standard Spring Security tables

**Cons:**
- Fixed table structure
- Limited flexibility
- Difficult to add custom user fields

### 5. Custom UserDetailsService

The most flexible approach - implement your own `UserDetailsService` with custom entity structure.

**Step 1: Create User Entity**

```java
@Entity
@Table(name = "users")
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(unique = true, nullable = false)
    private String email;

    @Column(nullable = false)
    private String password;

    @Column(nullable = false)
    private String fullName;

    private boolean enabled = true;

    private boolean accountNonLocked = true;

    @ElementCollection(fetch = FetchType.EAGER)
    @CollectionTable(name = "user_roles", joinColumns = @JoinColumn(name = "user_id"))
    @Column(name = "role")
    private Set<String> roles = new HashSet<>();

    @CreationTimestamp
    private Instant createdAt;

    @UpdateTimestamp
    private Instant updatedAt;

    // Constructors, getters, setters
}
```

**Step 2: Create Repository**

```java
public interface UserRepository extends JpaRepository<User, Long> {
    Optional<User> findByEmail(String email);
    boolean existsByEmail(String email);
}
```

**Step 3: Implement Custom UserDetails**

```java
public class CustomUserDetails implements UserDetails {

    private final User user;

    public CustomUserDetails(User user) {
        this.user = user;
    }

    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        return user.getRoles().stream()
            .map(role -> new SimpleGrantedAuthority("ROLE_" + role))
            .collect(Collectors.toList());
    }

    @Override
    public String getPassword() {
        return user.getPassword();
    }

    @Override
    public String getUsername() {
        return user.getEmail();
    }

    @Override
    public boolean isAccountNonExpired() {
        return true;
    }

    @Override
    public boolean isAccountNonLocked() {
        return user.isAccountNonLocked();
    }

    @Override
    public boolean isCredentialsNonExpired() {
        return true;
    }

    @Override
    public boolean isEnabled() {
        return user.isEnabled();
    }

    // Additional methods to access User entity
    public String getFullName() {
        return user.getFullName();
    }

    public Long getId() {
        return user.getId();
    }
}
```

**Step 4: Implement UserDetailsService**

```java
@Service
public class CustomUserDetailsService implements UserDetailsService {

    private final UserRepository userRepository;
    
    //constructor

    @Override
    public UserDetails loadUserByUsername(String email)
            throws UsernameNotFoundException {

        User user = userRepository.findByEmail(email)
            .orElseThrow(() -> new UsernameNotFoundException(
                "User not found with email: " + email));

        return new CustomUserDetails(user);
    }
}
```

**Step 5: Security Configuration**

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
    
    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/", "/register", "/css/**", "/js/**").permitAll()
                .requestMatchers("/admin/**").hasRole("ADMIN")
                .requestMatchers("/user/**").hasAnyRole("USER", "ADMIN")
                .anyRequest().authenticated()
            )
            .formLogin(form -> form
                .loginPage("/login")
                .loginProcessingUrl("/perform-login")
                .defaultSuccessUrl("/dashboard", true)
                .failureUrl("/login?error=true")
                .usernameParameter("email")  // Custom field name
                .passwordParameter("password")
                .permitAll()
            )
            .logout(logout -> logout
                .logoutUrl("/logout")
                .logoutSuccessUrl("/login?logout=true")
                .invalidateHttpSession(true)
                .deleteCookies("JSESSIONID")
                .permitAll()
            )
            .exceptionHandling(ex -> ex
                .accessDeniedPage("/access-denied")
            );

        return http.build();
    }

    
}
```

**Step 6: User Registration Service**

```java
@Service
public class UserService {

    @Autowired
    private UserRepository userRepository;

    @Autowired
    private PasswordEncoder passwordEncoder;

    public User registerNewUser(UserRegistrationDto dto) {
        // Check if user already exists
        if (userRepository.existsByEmail(dto.getEmail())) {
            throw new RuntimeException("Email already exists");
        }

        // Create new user
        User user = new User();
        user.setEmail(dto.getEmail());
        user.setPassword(passwordEncoder.encode(dto.getPassword()));
        user.setFullName(dto.getFullName());
        user.setEnabled(true);
        user.getRoles().add("USER");

        return userRepository.save(user);
    }

    public User getCurrentUser() {
        Authentication auth = SecurityContextHolder.getContext().getAuthentication();
        if (auth == null || !auth.isAuthenticated()) {
            return null;
        }

        CustomUserDetails userDetails = (CustomUserDetails) auth.getPrincipal();
        return userRepository.findById(userDetails.getId())
            .orElse(null);
    }
}
```

**Step 7: Controller Example**

```java
@Controller
public class AuthController {

    @Autowired
    private UserService userService;

    @GetMapping("/login")
    public String loginPage() {
        return "login";
    }

    @GetMapping("/register")
    public String registerPage(Model model) {
        model.addAttribute("user", new UserRegistrationDto());
        return "register";
    }

    @PostMapping("/register")
    public String registerUser(@Valid @ModelAttribute UserRegistrationDto dto,
                              BindingResult result) {
        if (result.hasErrors()) {
            return "register";
        }

        try {
            userService.registerNewUser(dto);
            return "redirect:/login?registered=true";
        } catch (Exception e) {
            result.rejectValue("email", "error.email", e.getMessage());
            return "register";
        }
    }

    @GetMapping("/dashboard")
    public String dashboard(Model model) {
        User user = userService.getCurrentUser();
        model.addAttribute("user", user);
        return "dashboard";
    }
}
```

**Accessing Current User in Controllers:**

```java
@RestController
@RequestMapping("/api")
public class ApiController {

    // Method 1: Using SecurityContextHolder
    @GetMapping("/user/info")
    public Map<String, Object> getUserInfo() {
        Authentication auth = SecurityContextHolder.getContext().getAuthentication();
        CustomUserDetails userDetails = (CustomUserDetails) auth.getPrincipal();

        return Map.of(
            "email", userDetails.getUsername(),
            "fullName", userDetails.getFullName(),
            "roles", userDetails.getAuthorities()
        );
    }

    // Method 2: Using @AuthenticationPrincipal annotation
    @GetMapping("/user/profile")
    public Map<String, Object> getProfile(
            @AuthenticationPrincipal CustomUserDetails userDetails) {

        return Map.of(
            "email", userDetails.getUsername(),
            "fullName", userDetails.getFullName()
        );
    }

    // Method 3: Using Authentication parameter
    @GetMapping("/user/details")
    public Map<String, Object> getDetails(Authentication authentication) {
        CustomUserDetails userDetails = (CustomUserDetails) authentication.getPrincipal();

        return Map.of(
            "name", authentication.getName(),
            "authorities", authentication.getAuthorities()
        );
    }
}
```

**Pros:**
- Complete flexibility
- Custom user fields
- Production-ready
- Can integrate with any data source

**Cons:**
- More code to write
- Need to implement all security logic

### 6. Spring Security Logout

Spring Security provides built-in logout functionality.

**Basic Logout Configuration:**

```java
@Bean
public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
    http
        .logout(logout -> logout
            .logoutUrl("/logout")                    // URL to trigger logout
            .logoutSuccessUrl("/login?logout=true")   // Redirect after logout
            .invalidateHttpSession(true)              // Invalidate session
            .deleteCookies("JSESSIONID")              // Delete cookies
            .clearAuthentication(true)                // Clear authentication
            .permitAll()
        );

    return http.build();
}
```

**Logout Form (GET request):**

```html
<form action="/logout" method="post">
    <input type="hidden" name="${_csrf.parameterName}" value="${_csrf.token}"/>
    <button type="submit">Logout</button>
</form>
```

**Logout Link (using Thymeleaf):**

```html
<form th:action="@{/logout}" method="post">
    <button type="submit">Logout</button>
</form>
```

**Custom Logout Handler:**

```java
@Component
public class CustomLogoutHandler implements LogoutHandler {

    @Override
    public void logout(HttpServletRequest request,
                      HttpServletResponse response,
                      Authentication authentication) {

        // Custom logout logic
        if (authentication != null) {
            String username = authentication.getName();
            System.out.println("User " + username + " logged out");

            // Log to database, send notification, etc.
        }
    }
}
```

**Custom Logout Success Handler:**

```java
@Component
public class CustomLogoutSuccessHandler implements LogoutSuccessHandler {

    @Override
    public void onLogoutSuccess(HttpServletRequest request,
                               HttpServletResponse response,
                               Authentication authentication)
            throws IOException, ServletException {

        if (authentication != null) {
            String username = authentication.getName();
            System.out.println("User " + username + " successfully logged out");
        }

        response.setStatus(HttpStatus.OK.value());
        response.sendRedirect("/login?logout=true");
    }
}
```

**Using Custom Handlers:**

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Autowired
    private CustomLogoutHandler customLogoutHandler;

    @Autowired
    private CustomLogoutSuccessHandler customLogoutSuccessHandler;

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
            .logout(logout -> logout
                .logoutUrl("/logout")
                .addLogoutHandler(customLogoutHandler)
                .logoutSuccessHandler(customLogoutSuccessHandler)
                .permitAll()
            );

        return http.build();
    }
}
```

**Logout Best Practices:**

1. **Always use POST** for logout (prevent CSRF attacks)
2. **Invalidate session** to free server resources
3. **Clear authentication** from SecurityContext
4. **Delete cookies** to remove client-side data
5. **Log logout events** for audit trails
6. **Redirect appropriately** based on user type


## Method-Level Security

Method-level security allows you to protect individual methods in your application using annotations. 
This provides fine-grained access control at the service or controller layer.

### Enabling Method Security

To use method-level security, you need to enable it in your configuration:

```java
@Configuration
@EnableWebSecurity
@EnableMethodSecurity  // Enable method-level security
public class SecurityConfig {

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(auth -> auth
                .anyRequest().authenticated()
            )
            .formLogin(Customizer.withDefaults());

        return http.build();
    }
}
```

### Method Security Annotations

Spring Security provides several annotations for method-level security:

#### 1. @PreAuthorize

Checks authorization **before** method execution. Most commonly used annotation.

**Basic Example:**
```java
@Service
public class UserService {

    @PreAuthorize("hasRole('ADMIN')")
    public void deleteUser(Long userId) {
        // Only admins can delete users
        userRepository.deleteById(userId);
    }

    @PreAuthorize("hasRole('USER')")
    public User getUserProfile(Long userId) {
        // Only authenticated users with USER role can access
        return userRepository.findById(userId).orElseThrow();
    }

    @PreAuthorize("hasAnyRole('USER', 'ADMIN')")
    public List<User> getAllUsers() {
        // Users with either USER or ADMIN role can access
        return userRepository.findAll();
    }
}
```

**Advanced Example with SpEL (Spring Expression Language):**
```java
@Service
public class PostService {

    @Autowired
    private PostRepository postRepository;

    // Allow access only to the post owner or admin
    @PreAuthorize("hasRole('ADMIN') or #userId == authentication.principal.id")
    public void updatePost(Long postId, Long userId, String content) {
        Post post = postRepository.findById(postId).orElseThrow();
        post.setContent(content);
        postRepository.save(post);
    }

    // Check if user is owner of the post
    @PreAuthorize("@postSecurityService.isOwner(#postId, authentication.principal.id)")
    public void deletePost(Long postId) {
        postRepository.deleteById(postId);
    }

    // Multiple conditions
    @PreAuthorize("hasRole('PREMIUM_USER') and #userId == authentication.principal.id")
    public void accessPremiumContent(Long userId) {
        // Only premium users can access their own premium content
    }
}
```

**Custom Security Service:**
```java
@Service("postSecurityService")
public class PostSecurityService {

    @Autowired
    private PostRepository postRepository;

    public boolean isOwner(Long postId, Long userId) {
        return postRepository.findById(postId)
            .map(post -> post.getUserId().equals(userId))
            .orElse(false);
    }

    public boolean canEdit(Long postId, Long userId) {
        Post post = postRepository.findById(postId).orElse(null);
        if (post == null) return false;

        // Owner can edit, or admin, or if post is in draft status
        return post.getUserId().equals(userId) ||
               post.getStatus().equals("DRAFT");
    }
}
```

#### 2. @PostAuthorize

Checks authorization **after** method execution. Useful when you need to check the return value.

```java
@Service
public class DocumentService {

    // Only allow if returned document belongs to current user
    @PostAuthorize("returnObject.userId == authentication.principal.id")
    public Document getDocument(Long documentId) {
        return documentRepository.findById(documentId).orElseThrow();
    }

    // Allow if user is admin or owns the returned document
    @PostAuthorize("hasRole('ADMIN') or returnObject.userId == authentication.principal.id")
    public Document getDocumentDetails(Long documentId) {
        return documentRepository.findById(documentId).orElseThrow();
    }
}
```

#### 3. @PreFilter

Filters collection parameters **before** method execution.

```java
@Service
public class BatchService {

    // Filter the list, keeping only items where userId matches authenticated user
    @PreFilter("filterObject.userId == authentication.principal.id")
    public void processOrders(List<Order> orders) {
        // Only process orders that belong to the current user
        orders.forEach(order -> orderRepository.save(order));
    }

    // Another example
    @PreFilter("hasRole('ADMIN') or filterObject.ownerId == authentication.principal.id")
    public void deleteMultipleDocuments(List<Document> documents) {
        documents.forEach(doc -> documentRepository.delete(doc));
    }
}
```

#### 4. @PostFilter

Filters collection return values **after** method execution.

```java
@Service
public class PostService {

    // Filter results to only show posts from user's friends or public posts
    @PostFilter("filterObject.isPublic == true or filterObject.userId == authentication.principal.id")
    public List<Post> getAllPosts() {
        return postRepository.findAll();
    }

    // Only return documents owned by current user
    @PostFilter("filterObject.ownerId == authentication.principal.id")
    public List<Document> getAllDocuments() {
        return documentRepository.findAll();
    }

    // Admin sees all, users see only their own
    @PostFilter("hasRole('ADMIN') or filterObject.userId == authentication.principal.id")
    public List<Order> getAllOrders() {
        return orderRepository.findAll();
    }
}
```

### Performance Considerations

```java
@Service
public class PerformanceExampleService {

    // ❌ Bad: Loads all records then filters
    @PostFilter("filterObject.userId == authentication.principal.id")
    public List<Order> getAllOrders() {
        return orderRepository.findAll(); // Loads all orders
    }

    // ✅ Good: Filters at database level
    @PreAuthorize("isAuthenticated()")
    public List<Order> getUserOrders(Long userId) {
        return orderRepository.findByUserId(userId); // Filters in SQL
    }
}
```

### Combining URL-Level and Method-Level Security

You can use both URL-level and method-level security together for defense in depth:

```java
@RestController
@RequestMapping("/api/posts")
public class PostController {

    @Autowired
    private PostService postService;

    // URL-level: Requires authentication
    // Method-level: Requires ADMIN role
    @PreAuthorize("hasRole('ADMIN')")
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deletePost(@PathVariable Long id) {
        postService.deletePost(id);
        return ResponseEntity.ok().build();
    }

    // Only post owner or admin can update
    @PreAuthorize("@postSecurityService.isOwner(#id, authentication.principal.id) or hasRole('ADMIN')")
    @PutMapping("/{id}")
    public ResponseEntity<Post> updatePost(
            @PathVariable Long id,
            @RequestBody PostUpdateDto dto) {

        Post updated = postService.updatePost(id, dto);
        return ResponseEntity.ok(updated);
    }

    // Anyone authenticated can view
    @GetMapping("/{id}")
    public ResponseEntity<Post> getPost(@PathVariable Long id) {
        Post post = postService.getPost(id);
        return ResponseEntity.ok(post);
    }
}
```

### SpEL Expressions in Method Security

Spring Expression Language (SpEL) provides powerful expressions for complex security rules:

#### Common SpEL Variables:

| Variable         | Description                          | Example                                |
|------------------|--------------------------------------|----------------------------------------|
| `authentication` | Current Authentication object        | `authentication.principal.id`          |
| `principal`      | Current user principal               | `principal.username`                   |
| `#paramName`     | Method parameter                     | `#userId == principal.id`              |
| `returnObject`   | Method return value (@PostAuthorize) | `returnObject.userId == principal.id`  |
| `filterObject`   | Current item in collection (filters) | `filterObject.ownerId == principal.id` |

#### SpEL Examples:

```java
@Service
public class SecurityExamplesService {

    // Check if parameter matches current user
    @PreAuthorize("#userId == authentication.principal.id")
    public void updateProfile(Long userId, ProfileDto dto) { }

    // Check if user has specific authority
    @PreAuthorize("hasAuthority('WRITE_PRIVILEGE')")
    public void writeData() { }

    // Multiple conditions with AND
    @PreAuthorize("hasRole('USER') and #userId == authentication.principal.id")
    public void updateUserData(Long userId) { }

    // Multiple conditions with OR
    @PreAuthorize("hasRole('ADMIN') or #ownerId == authentication.principal.id")
    public void deleteResource(Long ownerId) { }

    // Check authentication status
    @PreAuthorize("isAuthenticated()")
    public void authenticatedOnly() { }

    // Allow anonymous access
    @PreAuthorize("permitAll()")
    public void publicAccess() { }

    // Deny all access
    @PreAuthorize("denyAll()")
    public void noAccess() { }

    // Complex expression
    @PreAuthorize("hasRole('ADMIN') or (hasRole('USER') and #userId == authentication.principal.id)")
    public void complexCheck(Long userId) { }

    // Call custom bean method
    @PreAuthorize("@customSecurityService.hasAccess(#resourceId, authentication.principal.id)")
    public void checkCustomAccess(Long resourceId) { }
}
```

### Method Security with Custom Annotations

Create custom security annotations for reusability:

```java
@Target({ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@PreAuthorize("hasRole('ADMIN')")
public @interface IsAdmin {
}

@Target({ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@PreAuthorize("#userId == authentication.principal.id or hasRole('ADMIN')")
public @interface IsOwnerOrAdmin {
}

// Usage
@Service
public class CustomAnnotationService {

    @IsAdmin
    public void adminOnlyMethod() {
        // Only admins can access
    }

    @IsOwnerOrAdmin
    public void ownerOrAdminMethod(Long userId) {
        // Owner or admin can access
    }
}
```

### Exception Handling

When authorization fails, Spring Security throws `AccessDeniedException`. 
Handle it with a global exception handler:

```java
@RestControllerAdvice
public class SecurityExceptionHandler {

    @ExceptionHandler(AccessDeniedException.class)
    public ResponseEntity<ErrorResponse> handleAccessDenied(AccessDeniedException ex) {
        ErrorResponse error = new ErrorResponse(
            HttpStatus.FORBIDDEN.value(),
            "Access Denied",
            "You don't have permission to access this resource"
        );
        return ResponseEntity.status(HttpStatus.FORBIDDEN).body(error);
    }

    @ExceptionHandler(AuthenticationException.class)
    public ResponseEntity<ErrorResponse> handleAuthentication(AuthenticationException ex) {
        ErrorResponse error = new ErrorResponse(
            HttpStatus.UNAUTHORIZED.value(),
            "Authentication Failed",
            ex.getMessage()
        );
        return ResponseEntity.status(HttpStatus.UNAUTHORIZED).body(error);
    }
}
```

## Assignments

### Assignment 1: Basic Security Setup

**Objective:** Set up basic Spring Security with in-memory authentication

**Tasks:**
1. Create a new Spring Boot application with Spring Security
2. Configure three users: user, admin, manager with different roles
3. Create the following endpoints:
   - `/public/home` - accessible to everyone
   - `/user/profile` - accessible to USER and ADMIN
   - `/admin/dashboard` - accessible only to ADMIN
4. Test with different users and verify access control

**Expected Output:**
- Anonymous users can access `/public/home`
- USER can access `/user/profile` but not `/admin/dashboard`
- ADMIN can access both `/user/profile` and `/admin/dashboard`

### Assignment 2: Custom Login Page

**Objective:** Create a custom login page with styling

**Tasks:**
1. Create a custom login page (HTML/Thymeleaf)
2. Add CSS styling to make it attractive
3. Display error messages for failed login attempts
4. Show success message after registration

**Expected Output:**
- Custom styled login page
- Error messages displayed properly


### Assignment 3: Database Authentication

**Objective:** Implement JDBC-based authentication

**Tasks:**
1. Create database tables for users and authorities
2. Configure Spring Security to use JDBC authentication
3. Insert sample users via SQL scripts
4. Create a registration page to add new users
5. Hash passwords using BCrypt

**Expected Output:**
- Users stored in database
- New users can register
- Passwords are encrypted
- Login works with database users

### Assignment 4: Custom UserDetailsService

**Objective:** Implement custom user entity with JPA

**Tasks:**
1. Create a User entity with fields: id, email, password, fullName, enabled, roles
2. Create UserRepository with JPA
3. Implement CustomUserDetailsService
4. Create registration and login functionality
5. Add user profile page showing user details

**Expected Output:**
- Complete user management system
- Registration with validation
- Login with custom user entity
- Profile page showing user information

### Assignment 5: Role-Based Authorization

**Objective:** Implement complex role-based access control

**Tasks:**
1. Create roles: USER, PREMIUM_USER, MANAGER, ADMIN
2. Create the following endpoints:
   - `/free/content` - accessible to all authenticated users
   - `/premium/content` - accessible to PREMIUM_USER and above
   - `/manager/reports` - accessible to MANAGER and ADMIN
   - `/admin/settings` - accessible only to ADMIN

**Expected Output:**
- Different content for different roles
- Access denied page for unauthorized access

### Assignment 6: Logout and Session Management

**Objective:** Implement custom logout functionality

**Tasks:**
1. Create a custom logout handler to log logout events
2. Save login/logout timestamp in database
3. Display login history on user profile
4. Add session timeout configuration

**Expected Output:**
- Logout events logged in database
- User can see login history
- Session expires after inactivity

### Assignment 7: Password Management

**Objective:** Implement password change and reset functionality

**Tasks:**
1. Create "Change Password" page
2. Validate old password before changing
3. Implement password strength requirements
4. Create "Forgot Password" functionality (without email for now)

**Expected Output:**
- Users can change password
- Password validation working
- Password reset functionality

[Home](../README.md) | [← Previous: Spring Data JPA](80-spring-data-jpa.md) | [Next: Spring Security JWT →](100-spring-security-jwt.md)