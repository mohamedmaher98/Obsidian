# Spring Security JWT-based Authentication

## Table of Contents
- [Web Application Security vs API Security](#web-application-security-vs-api-security)
- [How JWT-Based Authentication Works](#how-jwt-based-authentication-works)
- [JWT Structure and Components](#jwt-structure-and-components)
- [JWT Authentication in Spring Security](#jwt-authentication-in-spring-security)
- [Generating RSA Keys with OpenSSL](#generating-rsa-keys-with-openssl)
- [OAuth2 Resource Server Configuration](#oauth2-resource-server-configuration)
- [Implementing JWT Token Provider](#implementing-jwt-token-provider)
- [Configuring Security Filters and CORS](#configuring-security-filters-and-cors)
- [Implementing Login Endpoint](#implementing-login-endpoint)
- [Retrieving Current User from JWT](#retrieving-current-user-from-jwt)
- [Best Practices and Security Considerations](#best-practices-and-security-considerations)
- [Assignments](#assignments)

## Web Application Security vs API Security

Understanding the difference between traditional web application security and 
API security is crucial for choosing the right authentication mechanism.

### Traditional Web Application Security

Traditional web applications are **stateful** and use server-side sessions.

**Characteristics:**

1. **Session-Based Authentication**
   - Server creates and stores session after login
   - Session ID stored in a cookie
   - Session data stored on server (in-memory or database)

2. **Cookie-Based**
   - JSESSIONID cookie sent with each request
   - Cookies are domain-specific
   - Automatic cookie management by browser

### REST API Security

Modern REST APIs are **stateless** and use token-based authentication.

**Characteristics:**

1. **Token-Based Authentication**
   - Server generates token after login
   - Token contains all user information
   - No server-side session storage

2. **Stateless**
   - Each request contains all necessary information
   - Server doesn't store client state
   - Easy to scale horizontally

3. **JSON Communication**
   - Data exchanged in JSON format
   - API consumed by multiple clients (web, mobile, third-party)

### When to Use What?

**Use Session-Based (Traditional):**
- Building a monolithic web application
- Server-side rendered pages
- Simple authentication requirements
- Need immediate session invalidation
- Single domain application

**Use Token-Based (JWT):**
- Building REST APIs
- Microservices architecture
- Mobile applications
- Single Page Applications (SPAs)
- Cross-domain authentication
- Need to scale horizontally

## How JWT-Based Authentication Works

JWT (JSON Web Token) is an open standard (RFC 7519) for securely transmitting information between parties as a JSON object.

### JWT Authentication Flow

Here's a complete authentication flow using JWT:

```
┌─────────┐                                        ┌─────────┐
│         │                                        │         │
│ Client  │                                        │  Server │
│         │                                        │         │
└────┬────┘                                        └────┬────┘
     │                                                  │
     │  1. POST /auth/login                             │
     │     { username, password }                       │
     ├─────────────────────────────────────────────────>│
     │                                                  │
     │                                          2. Validate credentials
     │                                                  │
     │                                          3. Generate JWT token
     │                                             (sign with secret key)
     │                                                  │
     │  4. Return JWT token                             │
     │     { token: "eyJhbGc..." }                      │
     │<─────────────────────────────────────────────────┤
     │                                                  │
5. Store token                                          │
   (localStorage/memory)                                │
     │                                                  │
     │  6. GET /api/users                               │
     │     Authorization: Bearer eyJhbGc...             │
     ├─────────────────────────────────────────────────>│
     │                                                  │
     │                                          7. Extract token
     │                                                  │
     │                                          8. Validate signature
     │                                                  │
     │                                          9. Verify expiration
     │                                                  │
     │                                          10. Extract user info
     │                                                  │
     │  11. Return protected resource                   │
     │      { users: [...] }                            │
     │<─────────────────────────────────────────────────┤
     │                                                  │
```

**Step-by-Step Explanation:**

1. **Client sends credentials** - User submits username and password to login endpoint

2. **Server validates credentials** - Server checks if username and password are correct

3. **Server generates JWT** - If valid, server creates a JWT token containing:
   - User ID
   - Username
   - Roles/Authorities
   - Expiration time
   - Other claims

4. **Server returns JWT** - Token sent back to client in response body

5. **Client stores token** - Client saves token (localStorage, sessionStorage, or in-memory)

6. **Client sends token** - For subsequent requests, client includes token in Authorization header

7. **Server extracts token** - Server reads the token from Authorization header

8. **Server validates signature** - Server verifies token hasn't been tampered with

9. **Server checks expiration** - Ensures token hasn't expired

10. **Server extracts user info** - Decodes token to get user details

11. **Server responds** - Returns requested resource if authorized

## JWT Structure and Components

A JWT consists of three parts separated by dots (`.`):

```
eyJhbGciOiJSUzI1NXXXX.eyJzdWIiOiIxjIzOTXXXX.signature_here

└──── Header ───────┘ └───── Payload ─────┘ └── Signature ──┘
```

### 1. Header

Contains metadata about the token:

```json
{
  "alg": "RS256",
  "typ": "JWT"
}
```

**Fields:**
- `alg` - Algorithm used for signing (RS256, HS256, etc.)
- `typ` - Type of token (JWT)

**Encoded:** `eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9`

### 2. Payload

Contains the claims (user information):

```json
{
  "sub": "user123",
  "name": "John Doe",
  "email": "john@example.com",
  "roles": ["USER", "ADMIN"],
  "iat": 1516239022,
  "exp": 1516242622
}
```

**Standard Claims:**

| Claim | Name               | Description                          |
|-------|--------------------|--------------------------------------|
| `sub` | Subject            | User identifier                      |
| `iss` | Issuer             | Who issued the token                 |
| `aud` | Audience           | Who the token is intended for        |
| `exp` | Expiration Time    | When token expires (Unix timestamp)  |
| `iat` | Issued At          | When token was created               |
| `nbf` | Not Before         | Token not valid before this time     |
| `jti` | JWT ID             | Unique identifier for the token      |

**Custom Claims:**

You can add custom claims:
```json
{
  "userId": 123,
  "username": "johndoe",
  "roles": ["USER", "ADMIN"],
  "permissions": ["read", "write"]
}
```

**Important:** Don't store sensitive data in payload (it's base64 encoded, not encrypted!)

**Encoded:** `eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiYWRtaW4iOnRydWUsImlhdCI6MTUxNjIzOTAyMn0`

### 3. Signature

Ensures token hasn't been tampered with:

```
RSASHA256(
  base64UrlEncode(header) + "." + base64UrlEncode(payload),
  privateKey
)
```

**How it works:**

1. Take encoded header and payload
2. Combine them with a dot
3. Sign with private key (RS256) or secret (HS256)
4. Append signature to token

**Verification:**

```
publicKey.verify(
  base64UrlEncode(header) + "." + base64UrlEncode(payload),
  signature
)
```

Server uses public key (RS256) or the same secret (HS256) to verify signature.

## JWT Authentication in Spring Security

Spring Security provides two approaches to implement JWT authentication:

### 1. Custom JWT Filter (Manual Approach)

Using libraries like **jjwt** (Java JWT):

**Dependency:**

```xml
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
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-jackson</artifactId>
    <version>0.11.5</version>
    <scope>runtime</scope>
</dependency>
```

**Pros:**
- Full control over JWT generation and validation
- Can customize claims easily
- Lightweight

**Cons:**
- More code to write
- Manual filter configuration
- Need to handle edge cases

### 2. OAuth2 Resource Server (Spring Security's Built-in)

Using **Spring Security OAuth2 Resource Server**:

**Dependency:**

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-oauth2-resource-server</artifactId>
</dependency>
```

**Pros:**
- Built-in Spring Security support
- Less code to write
- Well-tested and secure
- Follows OAuth2 standards
- Easy configuration

**Cons:**
- Requires understanding of OAuth2 concepts

**In this guide, we'll use OAuth2 Resource Server approach** as it's the recommended way.

## Generating RSA Keys with OpenSSL

For RS256 algorithm, we need a public-private key pair. Let's generate them using OpenSSL.

### Step 1: Generate Private Key

```bash
# Generate 2048-bit RSA private key
openssl genrsa -out private.pem 2048
```

**Output:**
```
Generating RSA private key, 2048 bit long modulus
.......+++
...........................................+++
e is 65537 (0x010001)
```

**private.pem:**
```
-----BEGIN RSA PRIVATE KEY-----
MIIEpAIBAAKCAQEA0Z5V3QJ5K... (many lines)
-----END RSA PRIVATE KEY-----
```

### Step 2: Generate Public Key

```bash
# Extract public key from private key
openssl rsa -in private.pem -pubout -out public.pem
```

**Output:**
```
writing RSA key
```

**public.pem:**
```
-----BEGIN PUBLIC KEY-----
MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEA0Z5V... (many lines)
-----END PUBLIC KEY-----
```

### Step 3: Convert to PKCS8 Format (Required for Java)

```bash
# Convert private key to PKCS8 format
openssl pkcs8 -topk8 -inform PEM -outform PEM -in private.pem -out private_key.pem -nocrypt
```

**private_key.pem:**
```
-----BEGIN PRIVATE KEY-----
MIIEvQIBADANBgkqhkiG9w0BAQEFAASC... (many lines)
-----END PRIVATE KEY-----
```

### Step 4: Place Keys in Spring Boot Project

Create a directory structure:

```
src/main/resources/
└── keys/
    ├── private_key.pem
    └── public.pem
```

### Alternative: One-Line Commands

```bash
# Generate private key in PKCS8 format directly
openssl genpkey -algorithm RSA -out private_key.pem -pkeyopt rsa_keygen_bits:2048

# Generate public key
openssl rsa -pubout -in private_key.pem -out public.pem
```

## OAuth2 Resource Server Configuration

Now let's configure Spring Security to use OAuth2 Resource Server with JWT.

### Step 1: Add Dependencies

**pom.xml:**

```xml
<dependencies>
    <!-- Spring Security -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-security</artifactId>
    </dependency>

    <!-- OAuth2 Resource Server -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-oauth2-resource-server</artifactId>
    </dependency>

    <!-- Spring Web -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

</dependencies>
```

### Step 2: Configure Application Properties

**application.properties:**

```properties

# JWT Configuration
jwt.private.key=classpath:keys/private_key.pem
jwt.public.key=classpath:keys/public.pem
jwt.expiration=3600000
```

### Step 3: Configure JwtDecoder and JwtEncoder

Create a configuration class to set up JWT encoding and decoding:

**RSAKeyProperties.java:**

```java
package com.example.security.config;

import org.springframework.boot.context.properties.ConfigurationProperties;

import java.security.interfaces.RSAPrivateKey;
import java.security.interfaces.RSAPublicKey;

@ConfigurationProperties(prefix = "jwt")
public record RSAKeyProperties(
    RSAPublicKey publicKey,
    RSAPrivateKey privateKey,
    Long expiration
) {
}
```

**JwtConfig.java:**

```java
package com.example.security.config;

import com.nimbusds.jose.jwk.JWK;
import com.nimbusds.jose.jwk.JWKSet;
import com.nimbusds.jose.jwk.RSAKey;
import com.nimbusds.jose.jwk.source.ImmutableJWKSet;
import com.nimbusds.jose.jwk.source.JWKSource;
import com.nimbusds.jose.proc.SecurityContext;
import org.springframework.boot.context.properties.EnableConfigurationProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.oauth2.jwt.JwtDecoder;
import org.springframework.security.oauth2.jwt.JwtEncoder;
import org.springframework.security.oauth2.jwt.NimbusJwtDecoder;
import org.springframework.security.oauth2.jwt.NimbusJwtEncoder;

@Configuration
@EnableConfigurationProperties(RSAKeyProperties.class)
public class JwtConfig {

    private final RSAKeyProperties rsaKeys;

    public JwtConfig(RSAKeyProperties rsaKeys) {
        this.rsaKeys = rsaKeys;
    }

    /**
     * JwtDecoder: Used to decode and validate JWT tokens
     * Uses the public key to verify the signature
     */
    @Bean
    public JwtDecoder jwtDecoder() {
        return NimbusJwtDecoder.withPublicKey(rsaKeys.publicKey()).build();
    }

    /**
     * JwtEncoder: Used to encode/generate JWT tokens
     * Uses the private key to sign the token
     */
    @Bean
    public JwtEncoder jwtEncoder() {
        JWK jwk = new RSAKey.Builder(rsaKeys.publicKey())
                .privateKey(rsaKeys.privateKey())
                .build();

        JWKSource<SecurityContext> jwks = new ImmutableJWKSet<>(new JWKSet(jwk));
        return new NimbusJwtEncoder(jwks);
    }
}
```

### Step 4: Configure JwtAuthenticationConverter

The `JwtAuthenticationConverter` converts a JWT token into an Authentication object with proper authorities.

Register a `JwtAuthenticationConverter` bean as follows:

```java
package com.example.security.config;

@Configuration
@EnableConfigurationProperties(RSAKeyProperties.class)
public class JwtConfig {

    //...
    
    @Bean
    JwtAuthenticationConverter jwtAuthenticationConverter() {
        var grantedAuthoritiesConverter = new JwtGrantedAuthoritiesConverter();
        grantedAuthoritiesConverter.setAuthoritiesClaimName("roles");
        grantedAuthoritiesConverter.setAuthorityPrefix("");

        var jwtAuthenticationConverter = new JwtAuthenticationConverter();
        jwtAuthenticationConverter.setJwtGrantedAuthoritiesConverter(grantedAuthoritiesConverter);
        return jwtAuthenticationConverter;
    }
}
```

## Implementing JWT Token Provider

Create a service to generate and validate JWT tokens.

**JwtTokenProvider.java:**

```java
package com.example.security.jwt;

import org.springframework.security.core.Authentication;
import org.springframework.security.core.GrantedAuthority;
import org.springframework.security.oauth2.jwt.JwtClaimsSet;
import org.springframework.security.oauth2.jwt.JwtEncoder;
import org.springframework.security.oauth2.jwt.JwtEncoderParameters;
import org.springframework.stereotype.Service;

import java.time.Instant;
import java.time.temporal.ChronoUnit;
import java.util.stream.Collectors;

@Service
public class JwtTokenProvider {

    private final JwtEncoder jwtEncoder;
    private final Long jwtExpiration;

    public JwtTokenProvider(JwtEncoder jwtEncoder) {
        this.jwtEncoder = jwtEncoder;
        this.jwtExpiration = 3600000L; // read from config
    }

    /**
     * Generate JWT token from Authentication object
     */
    public String generateToken(Authentication authentication) {
        Instant now = Instant.now();

        // Extract roles/authorities from authentication
        String roles = authentication.getAuthorities().stream()
                .map(GrantedAuthority::getAuthority)
                .collect(Collectors.joining(" "));

        // Build JWT claims
        JwtClaimsSet claims = JwtClaimsSet.builder()
                .issuer("self")                                    // Token issuer
                .issuedAt(now)                                     // Issued time
                .expiresAt(now.plus(jwtExpiration, ChronoUnit.MILLIS))  // Expiration time
                .subject(authentication.getName())                 // Username
                .claim("roles", roles)                            // User roles
                .build();

        // Encode and return token
        return jwtEncoder.encode(JwtEncoderParameters.from(claims)).getTokenValue();
    }

    /**
     * Generate token with custom user details
     */
    public String generateToken(String username, String roles, Long userId) {
        Instant now = Instant.now();

        JwtClaimsSet claims = JwtClaimsSet.builder()
                .issuer("self")
                .issuedAt(now)
                .expiresAt(now.plus(jwtExpiration, ChronoUnit.MILLIS))
                .subject(username)
                .claim("roles", roles)
                .claim("userId", userId)
                .build();

        return jwtEncoder.encode(JwtEncoderParameters.from(claims)).getTokenValue();
    }

    /**
     * Generate refresh token (longer expiration)
     */
    public String generateRefreshToken(Authentication authentication) {
        Instant now = Instant.now();
        Long refreshExpiration = 604800000L; // 7 days

        JwtClaimsSet claims = JwtClaimsSet.builder()
                .issuer("self")
                .issuedAt(now)
                .expiresAt(now.plus(refreshExpiration, ChronoUnit.MILLIS))
                .subject(authentication.getName())
                .claim("type", "refresh")
                .build();

        return jwtEncoder.encode(JwtEncoderParameters.from(claims)).getTokenValue();
    }
}
```

## Configuring Security Filters and CORS

Configure Spring Security with JWT authentication, session management, CSRF, and CORS.

**SecurityConfig.java:**

```java
package com.example.security.config;

import com.example.security.jwt.JwtToAuthenticationConverter;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.http.HttpMethod;
import org.springframework.security.authentication.AuthenticationManager;
import org.springframework.security.authentication.ProviderManager;
import org.springframework.security.authentication.dao.DaoAuthenticationProvider;
import org.springframework.security.config.annotation.method.configuration.EnableMethodSecurity;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configurers.AbstractHttpConfigurer;
import org.springframework.security.config.http.SessionCreationPolicy;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.security.oauth2.server.resource.authentication.JwtAuthenticationProvider;
import org.springframework.security.oauth2.server.resource.web.BearerTokenAuthenticationEntryPoint;
import org.springframework.security.oauth2.server.resource.web.access.BearerTokenAccessDeniedHandler;
import org.springframework.security.web.SecurityFilterChain;
import org.springframework.web.cors.CorsConfiguration;
import org.springframework.web.cors.CorsConfigurationSource;
import org.springframework.web.cors.UrlBasedCorsConfigurationSource;

import java.util.List;

@Configuration
@EnableWebSecurity
@EnableMethodSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
                // Disable CSRF (not needed for stateless JWT authentication)
                .csrf(AbstractHttpConfigurer::disable)

                // Configure CORS
                .cors(cors -> cors.configurationSource(corsConfigurationSource()))

                // Configure URL authorization
                .authorizeHttpRequests(auth -> auth
                        // Public endpoints
                        .requestMatchers("/auth/**").permitAll()
                        .requestMatchers("/public/**").permitAll()
                        .requestMatchers(HttpMethod.GET, "/api/posts").permitAll()

                        // Admin endpoints
                        .requestMatchers("/admin/**").hasRole("ADMIN")

                        // All other endpoints require authentication
                        .anyRequest().authenticated()
                )

                // Configure OAuth2 Resource Server
                .oauth2ResourceServer(oauth2 -> oauth2.jwt(Customizer.withDefaults()))

                // Configure session management (stateless)
                .sessionManagement(session -> session
                        .sessionCreationPolicy(SessionCreationPolicy.STATELESS)
                )

                // Configure exception handling
                .exceptionHandling(ex -> ex
                        .authenticationEntryPoint(new BearerTokenAuthenticationEntryPoint())
                        .accessDeniedHandler(new BearerTokenAccessDeniedHandler())
                );

        return http.build();
    }

    /**
     * Configure CORS
     */
    @Bean
    public CorsConfigurationSource corsConfigurationSource() {
        CorsConfiguration configuration = new CorsConfiguration();

        // Allow specific origins (change for production) //make it configurable
        configuration.setAllowedOrigins(List.of("http://localhost:3000", "http://localhost:4200"));

        // Allow specific HTTP methods
        configuration.setAllowedMethods(List.of("GET", "POST", "PUT", "DELETE", "OPTIONS"));

        // Allow specific headers
        configuration.setAllowedHeaders(List.of("*"));

        // Allow credentials (cookies, authorization headers)
        configuration.setAllowCredentials(true);

        // Expose headers to client
        configuration.setExposedHeaders(List.of("Authorization"));

        // Max age for preflight requests
        configuration.setMaxAge(3600L);

        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        source.registerCorsConfiguration("/**", configuration);

        return source;
    }

    /**
     * Password encoder bean
     */
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }

    /**
     * Authentication Manager for username/password authentication
     */
    @Bean
    public AuthenticationManager authenticationManager(UserDetailsService uds, PasswordEncoder pe) {
        var authProvider = new DaoAuthenticationProvider(uds);
        authProvider.setPasswordEncoder(pe);
        return new ProviderManager(authProvider);
    }
}
```

### Configuration Explained

**1. CSRF Disabled:**
```java
.csrf(AbstractHttpConfigurer::disable)
```
- Not needed for stateless JWT authentication
- CSRF protects against cookie-based attacks
- JWT sent in Authorization header, not cookies

**2. Session Management:**
```java
.sessionManagement(session -> session
    .sessionCreationPolicy(SessionCreationPolicy.STATELESS)
)
```
- `STATELESS` - No session created
- Each request must contain JWT token
- Server doesn't store any session data

**3. OAuth2 Resource Server:**
```java
.oauth2ResourceServer(oauth2 -> oauth2.jwt(Customizer.withDefaults()))
```

- Configures JWT validation
- Uses `JwtDecoder` to validate tokens
- Converts JWT to Authentication using the registered JwtAuthenticationConverter

**4. Exception Handling:**
```java
.exceptionHandling(ex -> ex
    .authenticationEntryPoint(new BearerTokenAuthenticationEntryPoint())
    .accessDeniedHandler(new BearerTokenAccessDeniedHandler())
)
```
- `BearerTokenAuthenticationEntryPoint` - Returns 401 Unauthorized
- `BearerTokenAccessDeniedHandler` - Returns 403 Forbidden

**5. CORS Configuration:**
- Allows cross-origin requests from specified origins
- Required for SPAs (React, Angular, Vue) on different ports
- Allows specific HTTP methods and headers

## Implementing Login Endpoint

Create an authentication controller to handle login requests.

### Step 1: Create User Entity

**User.java:**

```java
package com.example.security.entity;

import jakarta.persistence.*;
import java.util.HashSet;
import java.util.Set;

@Entity
@Table(name = "users")
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(unique = true, nullable = false)
    private String username;

    @Column(nullable = false)
    private String password;

    @Column(unique = true, nullable = false)
    private String email;

    private boolean enabled = true;

    @ElementCollection(fetch = FetchType.EAGER)
    @CollectionTable(name = "user_roles", joinColumns = @JoinColumn(name = "user_id"))
    @Column(name = "role")
    private Set<String> roles = new HashSet<>();

    // Constructors, getters, setters
    public User() {}

    public User(String username, String password, String email) {
        this.username = username;
        this.password = password;
        this.email = email;
    }

    // Getters and Setters
}
```

### Step 2: Create Repository

**UserRepository.java:**

```java
package com.example.security.repository;

import com.example.security.entity.User;
import org.springframework.data.jpa.repository.JpaRepository;

import java.util.Optional;

public interface UserRepository extends JpaRepository<User, Long> {
    Optional<User> findByUsername(String username);
    boolean existsByUsername(String username);
    boolean existsByEmail(String email);
}
```

### Step 3: Implement UserDetailsService

**CustomUserDetailsService.java:**

```java
package com.example.security.service;

import com.example.security.entity.User;
import com.example.security.repository.UserRepository;
import org.springframework.security.core.GrantedAuthority;
import org.springframework.security.core.authority.SimpleGrantedAuthority;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.core.userdetails.UsernameNotFoundException;
import org.springframework.stereotype.Service;

import java.util.Collection;
import java.util.stream.Collectors;

@Service
public class CustomUserDetailsService implements UserDetailsService {

    private final UserRepository userRepository;

    public CustomUserDetailsService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        User user = userRepository.findByUsername(username)
                .orElseThrow(() -> new UsernameNotFoundException("User not found: " + username));

        return org.springframework.security.core.userdetails.User
                .withUsername(user.getUsername())
                .password(user.getPassword())
                .authorities(getAuthorities(user))
                .disabled(!user.isEnabled())
                .build();
    }

    private Collection<? extends GrantedAuthority> getAuthorities(User user) {
        return user.getRoles().stream()
                .map(role -> new SimpleGrantedAuthority("ROLE_" + role))
                .toList();
    }
}
```

### Step 4: Create DTOs

**LoginRequest.java:**

```java
package com.example.security.dto;

import jakarta.validation.constraints.NotBlank;

public record LoginRequest(
    @NotBlank(message = "Username is required")
    String username,

    @NotBlank(message = "Password is required")
    String password){
}
```

**LoginResponse.java:**

```java
package com.example.security.dto;

public record LoginResponse(
    String accessToken,
    String refreshToken,
    Long expiresIn){
}
```

**RegisterRequest.java:**

```java
package com.example.security.dto;

import jakarta.validation.constraints.Email;
import jakarta.validation.constraints.NotBlank;
import jakarta.validation.constraints.Size;

public record RegisterRequest(
    @NotBlank(message = "Username is required")
    @Size(min = 3, max = 50, message = "Username must be between 3 and 50 characters")
    String username,

    @NotBlank(message = "Email is required")
    @Email(message = "Email must be valid")
    String email,

    @NotBlank(message = "Password is required")
    @Size(min = 6, max = 100, message = "Password must be at least 6 characters")
    String password){
}
```

### Step 5: Create Authentication Controller

**AuthController.java:**

```java
package com.example.security.controller;

import com.example.security.dto.LoginRequest;
import com.example.security.dto.LoginResponse;
import com.example.security.dto.RegisterRequest;
import com.example.security.entity.User;
import com.example.security.jwt.JwtTokenProvider;
import com.example.security.repository.UserRepository;
import jakarta.validation.Valid;
import org.springframework.http.ResponseEntity;
import org.springframework.security.authentication.AuthenticationManager;
import org.springframework.security.authentication.UsernamePasswordAuthenticationToken;
import org.springframework.security.core.Authentication;
import org.springframework.security.core.context.SecurityContextHolder;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.web.bind.annotation.*;

import java.util.HashMap;
import java.util.Map;

@RestController
@RequestMapping("/api/auth")
public class AuthController {

    private final AuthenticationManager authenticationManager;
    private final UserRepository userRepository;
    private final PasswordEncoder passwordEncoder;
    private final JwtTokenProvider jwtTokenProvider;

    public AuthController(AuthenticationManager authenticationManager,
                          UserRepository userRepository,
                          PasswordEncoder passwordEncoder,
                          JwtTokenProvider jwtTokenProvider) {
        this.authenticationManager = authenticationManager;
        this.userRepository = userRepository;
        this.passwordEncoder = passwordEncoder;
        this.jwtTokenProvider = jwtTokenProvider;
    }

    /**
     * Login endpoint - Returns JWT token
     */
    @PostMapping("/login")
    public ResponseEntity<?> login(@Valid @RequestBody LoginRequest loginRequest) {
        try {
            // Authenticate user
            Authentication authentication = authenticationManager.authenticate(
                    new UsernamePasswordAuthenticationToken(
                            loginRequest.username(),
                            loginRequest.password()
                    )
            );

            // Set authentication in security context
            SecurityContextHolder.getContext().setAuthentication(authentication);

            // Generate JWT tokens
            String accessToken = jwtTokenProvider.generateToken(authentication);
            String refreshToken = jwtTokenProvider.generateRefreshToken(authentication);

            // Return tokens
            LoginResponse response = new LoginResponse(accessToken, refreshToken, 3600000L);
            return ResponseEntity.ok(response);

        } catch (Exception e) {
            Map<String, String> error = new HashMap<>();
            error.put("error", "Invalid username or password");
            return ResponseEntity.badRequest().body(error);
        }
    }

    /**
     * Register endpoint - Creates new user
     */
    @PostMapping("/register")
    public ResponseEntity<?> register(@Valid @RequestBody RegisterRequest registerRequest) {
        // Check if username exists
        if (userRepository.existsByUsername(registerRequest.username())) {
            Map<String, String> error = new HashMap<>();
            error.put("error", "Username already exists");
            return ResponseEntity.badRequest().body(error);
        }

        // Check if email exists
        if (userRepository.existsByEmail(registerRequest.email())) {
            Map<String, String> error = new HashMap<>();
            error.put("error", "Email already exists");
            return ResponseEntity.badRequest().body(error);
        }

        // Create new user
        User user = new User();
        user.setUsername(registerRequest.username());
        user.setEmail(registerRequest.email());
        user.setPassword(passwordEncoder.encode(registerRequest.password()));
        user.getRoles().add("USER");

        userRepository.save(user);

        Map<String, String> response = new HashMap<>();
        response.put("message", "User registered successfully");
        return ResponseEntity.ok(response);
    }
}
```

### Step 6: Test with cURL

**Register a new user:**

```bash
curl -X POST http://localhost:8080/auth/register \
  -H "Content-Type: application/json" \
  -d '{
    "username": "john",
    "email": "john@example.com",
    "password": "password123"
  }'
```

**Response:**
```json
{
  "message": "User registered successfully"
}
```

**Login:**

```bash
curl -X POST http://localhost:8080/auth/login \
  -H "Content-Type: application/json" \
  -d '{
    "username": "john",
    "password": "password123"
  }'
```

**Response:**
```json
{
  "accessToken": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...",
  "refreshToken": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...",
  "tokenType": "Bearer",
  "expiresIn": 3600000
}
```

## Retrieving Current User from JWT

Now let's implement endpoints to retrieve the current authenticated user's details from the JWT token.

### Step 1: Create UserController

**UserController.java:**

```java
package com.example.security.controller;

import com.example.security.entity.User;
import com.example.security.repository.UserRepository;
import org.springframework.http.ResponseEntity;
import org.springframework.security.core.Authentication;
import org.springframework.security.core.context.SecurityContextHolder;
import org.springframework.security.oauth2.jwt.Jwt;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.HashMap;
import java.util.Map;

@RestController
@RequestMapping("/api/user")
public class UserController {

    private final UserRepository userRepository;

    public UserController(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    /**
     * Get current user details from JWT token
     * Method 1: Using SecurityContextHolder
     */
    @GetMapping("/me")
    public ResponseEntity<?> getCurrentUser() {
        // Get authentication from security context
        Authentication authentication = SecurityContextHolder.getContext().getAuthentication();

        if (authentication == null || !authentication.isAuthenticated()) {
            return ResponseEntity.status(401).body("Not authenticated");
        }

        // Get JWT from authentication
        Jwt jwt = (Jwt) authentication.getPrincipal();

        // Extract user details from JWT
        String username = jwt.getSubject();
        String roles = jwt.getClaimAsString("roles");

        // Fetch user from database
        User user = userRepository.findByUsername(username)
                .orElseThrow(() -> new RuntimeException("User not found"));

        // Create response
        Map<String, Object> response = new HashMap<>();
        response.put("id", user.getId());
        response.put("username", user.getUsername());
        response.put("email", user.getEmail());
        response.put("roles", user.getRoles());

        return ResponseEntity.ok(response);
    }

    /**
     * Method 2: Using Authentication parameter
     */
    @GetMapping("/profile")
    public ResponseEntity<?> getUserProfile(Authentication authentication) {
        if (authentication == null) {
            return ResponseEntity.status(401).body("Not authenticated");
        }

        Jwt jwt = (Jwt) authentication.getPrincipal();
        String username = jwt.getSubject();

        User user = userRepository.findByUsername(username)
                .orElseThrow(() -> new RuntimeException("User not found"));

        Map<String, Object> response = new HashMap<>();
        response.put("id", user.getId());
        response.put("username", user.getUsername());
        response.put("email", user.getEmail());
        response.put("roles", user.getRoles());
        response.put("enabled", user.isEnabled());

        return ResponseEntity.ok(response);
    }

    /**
     * Get all JWT claims
     */
    @GetMapping("/claims")
    public ResponseEntity<?> getAllClaims(Authentication authentication) {
        Jwt jwt = (Jwt) authentication.getPrincipal();

        Map<String, Object> claims = new HashMap<>();
        claims.put("subject", jwt.getSubject());
        claims.put("issuer", jwt.getIssuer());
        claims.put("issuedAt", jwt.getIssuedAt());
        claims.put("expiresAt", jwt.getExpiresAt());
        claims.put("roles", jwt.getClaimAsString("roles"));
        claims.put("allClaims", jwt.getClaims());

        return ResponseEntity.ok(claims);
    }
}
```

### Step 2: Test with cURL

**Get current user (requires JWT):**

```bash
curl -X GET http://localhost:8080/api/user/me \
  -H "Authorization: Bearer eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9..."
```

**Response:**
```json
{
  "id": 1,
  "username": "john",
  "email": "john@example.com",
  "roles": ["USER"]
}
```

**Get JWT claims:**

```bash
curl -X GET http://localhost:8080/api/user/claims \
  -H "Authorization: Bearer eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9..."
```

**Response:**
```json
{
  "subject": "john",
  "issuer": "self",
  "issuedAt": "2024-01-15T10:30:00Z",
  "expiresAt": "2024-01-15T11:30:00Z",
  "roles": "ROLE_USER",
  "allClaims": {
    "sub": "john",
    "iss": "self",
    "iat": 1705318200,
    "exp": 1705321800,
    "roles": "ROLE_USER"
  }
}
```

## Best Practices and Security Considerations

### 1. Token Expiration

**Keep access tokens short-lived:**

```java
// Access token: 15-60 minutes
jwt.expiration=900000  // 15 minutes

// Refresh token: 7-30 days
jwt.refresh.expiration=604800000  // 7 days
```

**Implement refresh token mechanism:**

```java
@PostMapping("/refresh")
public ResponseEntity<?> refreshToken(@RequestBody Map<String, String> request) {
    String refreshToken = request.get("refreshToken");

    // Validate refresh token
    // Generate new access token

    return ResponseEntity.ok(new LoginResponse(newAccessToken, refreshToken, 3600000L));
}
```

### 2. Token Storage

**Client-side storage options:**

```javascript
// ✅ Good: In-memory (most secure)
let token = null;

function setToken(value) {
    token = value;
}

// ❌ Bad: localStorage (vulnerable to XSS)
localStorage.setItem('token', token);

// ⚠️ Okay: sessionStorage (better than localStorage)
sessionStorage.setItem('token', token);

// ✅ Best: httpOnly cookie (if using same domain)
// Set cookie from backend with httpOnly flag
```

### 3. Secure Key Management

**Never hardcode keys:**

```java
// ❌ Bad
String secret = "mySecretKey123";

// ✅ Good - Use environment variables
@Value("${jwt.private.key}")
private String privateKeyPath;

// ✅ Better - Use secret management services
// AWS Secrets Manager, Azure Key Vault, etc.
```

## Assignments

### Assignment 1: Basic JWT Authentication

**Objective:** Implement JWT authentication from scratch

**Tasks:**
1. Generate RSA keys using OpenSSL
2. Configure Spring Security with OAuth2 Resource Server
3. Implement login endpoint that returns JWT
4. Create a protected endpoint that requires JWT
5. Test with Postman or cURL

**Expected Output:**
- User can register and login
- JWT token returned on successful login
- Protected endpoints accessible only with valid JWT

### Assignment 2: User Profile Management

**Objective:** Implement user profile CRUD operations

**Tasks:**
1. Create endpoints to get current user profile
2. Implement update profile endpoint (only own profile)
3. Add change password functionality
4. Implement delete account endpoint

**Expected Output:**
- Users can view and update their profile
- Users can change their password
- Users can delete their account

### Assignment 3: Role-Based Access Control

**Objective:** Implement role-based authorization with JWT

**Tasks:**
1. Create USER and ADMIN roles
2. Implement admin-only endpoints
3. Use @PreAuthorize for method-level security
4. Create endpoints that require specific roles

**Expected Output:**
- Different access levels based on roles
- Admin can access all endpoints
- Regular users have limited access

### Assignment 4: Refresh Token Implementation

**Objective:** Implement refresh token mechanism

**Tasks:**
1. Generate refresh tokens (longer expiration)
2. Store refresh tokens in database
3. Implement /auth/refresh endpoint
4. Validate refresh tokens
5. Return new access token

**Expected Output:**
- Users get both access and refresh tokens on login
- Access token can be refreshed without re-login
- Refresh tokens can be invalidated

### Assignment 5: Post Management with JWT

**Objective:** Apply JWT security to jblogger REST API endpoints

**Tasks:**
1. Generate RSA keys using OpenSSL
2. Configure Spring Security with OAuth2 Resource Server
3. Secure endpoints with `SecurityFilterChain` and/or `@PreAuthorize`
4. Test JWT security with Postman

**Expected Output:**
- JWT security applied to jblogger endpoints
- Access to protected endpoints requires valid JWT

[Home](../README.md) | [← Previous: Spring Security Basics](90-spring-security-basics.md) | [Next: Testing →](110-testing.md)