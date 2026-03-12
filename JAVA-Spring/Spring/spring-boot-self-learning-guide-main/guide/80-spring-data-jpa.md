# Spring Data JPA

## Table of Contents
- [Main Features of Spring Data JPA](#main-features-of-spring-data-jpa)
- [Setup](#setup)
- [Creating the Entity](#creating-the-entity)
- [CRUD Operations with Spring Data JPA](#crud-operations-with-spring-data-jpa)
- [Sorting and Pagination](#sorting-and-pagination)
- [Custom Finder Methods](#custom-finder-methods)
- [Spring Data JPA Projections](#spring-data-jpa-projections)
- [Spring Data JPA Auditing](#spring-data-jpa-auditing)
- [Summary](#summary)
- [Assignment](#assignment)

**Spring Data JPA** is a part of the Spring Data family that simplifies data access using the Java Persistence API (JPA). 
It reduces boilerplate code by providing repository abstractions and eliminates the need to write repetitive CRUD operations manually.

## Main Features of Spring Data JPA

- **Automatic CRUD Operations**: Provides built-in methods like `save()`, `findById()`, `findAll()`, `delete()` without writing implementation code
- **Query Methods**: Automatically generates queries from method names (e.g., `findByTitle()`, `findByCreatedAtBefore()`)
- **Pagination and Sorting**: Built-in support for paginating and sorting query results
- **Custom Queries**: Write custom queries using JPQL or native SQL with `@Query` annotation
- **Auditing**: Automatically track creation and modification timestamps and users
- **Projections**: Fetch only specific fields instead of entire entities

In this guide, we'll use the **bookmarks** example from the Flyway section to demonstrate Spring Data JPA features.

## Setup

Add the Spring Data JPA dependency to your `pom.xml`:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
    <groupId>org.postgresql</groupId>
    <artifactId>postgresql</artifactId>
    <scope>runtime</scope>
</dependency>
```

Configure database connection in `application.properties`:

```properties
spring.datasource.url=jdbc:postgresql://localhost:5432/postgres
spring.datasource.username=postgres
spring.datasource.password=postgres

spring.jpa.show-sql=true
```

## Creating the Entity

Create a `Bookmark` entity class that maps to the `bookmarks` table:

```java
package com.example.demo.domain;

import jakarta.persistence.*;
import java.time.Instant;

@Entity
@Table(name = "bookmarks")
public class Bookmark {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false)
    private String title;

    @Column(nullable = false)
    private String url;

    @Column(name = "created_at")
    private Instant createdAt;

    // Constructors
    protected Bookmark() {
    }

    public Bookmark(String title, String url) {
        this.title = title;
        this.url = url;
        this.createdAt = Instant.now();
    }

    // Getters and Setters
}
```

**Key Annotations:**
- `@Entity`: Marks the class as a JPA entity
- `@Table`: Specifies the table name (optional if table name matches class name)
- `@Id`: Marks the primary key field
- `@GeneratedValue`: Specifies how the primary key is generated
- `@Column`: Maps the field to a database column

## CRUD Operations with Spring Data JPA

Create a repository interface by extending `JpaRepository`:

```java
package com.example.demo.repository;

import com.example.demo.domain.Bookmark;
import org.springframework.data.jpa.repository.JpaRepository;

public interface BookmarkRepository extends JpaRepository<Bookmark, Long> {
    // No code needed! Spring Data JPA provides implementations automatically
}
```

That's it! Spring Data JPA automatically provides the following methods:

- `save(entity)` - Create or update
- `findById(id)` - Find by ID
- `findAll()` - Find all entities
- `deleteById(id)` - Delete by ID
- `count()` - Count all entities
- And many more...

### Using the Repository

Create a service class to demonstrate CRUD operations:

```java
package com.example.demo.service;

import com.example.demo.domain.Bookmark;
import com.example.demo.repository.BookmarkRepository;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import java.util.List;
import java.util.Optional;

@Service
@Transactional
public class BookmarkService {

    private final BookmarkRepository bookmarkRepository;

    public BookmarkService(BookmarkRepository bookmarkRepository) {
        this.bookmarkRepository = bookmarkRepository;
    }

    // Create
    public Bookmark createBookmark(Bookmark bookmark) {
        return bookmarkRepository.save(bookmark);
    }

    // Read - Find by ID
    public Optional<Bookmark> getBookmarkById(Long id) {
        return bookmarkRepository.findById(id);
    }

    // Read - Find all
    public List<Bookmark> getAllBookmarks() {
        return bookmarkRepository.findAll();
    }

    // Update
    public Bookmark updateBookmark(Long id, Bookmark updatedBookmark) {
        return bookmarkRepository.findById(id)
            .map(bookmark -> {
                bookmark.setTitle(updatedBookmark.getTitle());
                bookmark.setUrl(updatedBookmark.getUrl());
                return bookmarkRepository.save(bookmark);
            })
            .orElseThrow(() -> new RuntimeException("Bookmark not found with id: " + id));
    }

    // Delete
    public void deleteBookmark(Long id) {
        bookmarkRepository.deleteById(id);
    }

    // Count
    public long countBookmarks() {
        return bookmarkRepository.count();
    }
}
```

## Sorting and Pagination

Spring Data JPA provides excellent support for sorting and pagination.

### Sorting

```java
import org.springframework.data.domain.Sort;

@Service
@Transactional
public class BookmarkService {

    private final BookmarkRepository bookmarkRepository;

    // ... constructor

    // Sort by title ascending
    public List<Bookmark> getAllBookmarksSortedByTitle() {
        return bookmarkRepository.findAll(Sort.by(Sort.Direction.ASC, "title"));
    }

    // Sort by created date descending
    public List<Bookmark> getAllBookmarksSortedByDate() {
        return bookmarkRepository.findAll(Sort.by(Sort.Direction.DESC, "createdAt"));
    }

    // Sort by multiple fields
    public List<Bookmark> getAllBookmarksSortedByMultipleFields() {
        Sort sort = Sort.by(
            Sort.Order.desc("createdAt"),
            Sort.Order.asc("title")
        );
        return bookmarkRepository.findAll(sort);
    }
}
```

### Pagination

```java
import org.springframework.data.domain.Page;
import org.springframework.data.domain.PageRequest;
import org.springframework.data.domain.Pageable;

@Service
@Transactional
public class BookmarkService {

    private final BookmarkRepository bookmarkRepository;

    // ... constructor

    // Get paginated results
    public Page<Bookmark> getBookmarksPaginated(int page, int size) {
        Pageable pageable = PageRequest.of(page, size);
        return bookmarkRepository.findAll(pageable);
    }

    // Pagination with sorting
    public Page<Bookmark> getBookmarksPaginatedAndSorted(int page, int size) {
        Pageable pageable = PageRequest.of(
            page,
            size,
            Sort.by(Sort.Direction.DESC, "createdAt")
        );
        return bookmarkRepository.findAll(pageable);
    }
}
```

The `Page` object contains useful information:
- `getContent()` - List of entities on current page
- `getTotalElements()` - Total number of elements
- `getTotalPages()` - Total number of pages
- `getNumber()` - Current page number
- `hasNext()` - Whether there is a next page
- `hasPrevious()` - Whether there is a previous page

### Using Pagination in a Controller

```java
package com.example.demo.controller;

import com.example.demo.domain.Bookmark;
import com.example.demo.service.BookmarkService;
import org.springframework.data.domain.Page;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/api/bookmarks")
public class BookmarkController {

    private final BookmarkService bookmarkService;

    public BookmarkController(BookmarkService bookmarkService) {
        this.bookmarkService = bookmarkService;
    }

    @GetMapping
    public Page<Bookmark> getBookmarks(
            @RequestParam(defaultValue = "0") int page,
            @RequestParam(defaultValue = "10") int size) {
        return bookmarkService.getBookmarksPaginatedAndSorted(page, size);
    }
}
```

Access the endpoint: `http://localhost:8080/api/bookmarks?page=0&size=10`

**NOTE:** Instead of directly returning a `Page<T>` object, map it to a custom DTO by exposing only the necessary fields.

## Custom Finder Methods

Spring Data JPA can automatically generate queries from method names.

### Using findBy...() Methods

Add these methods to your `BookmarkRepository`:

```java
package com.example.demo.repository;

import com.example.demo.domain.Bookmark;
import org.springframework.data.jpa.repository.JpaRepository;

import java.time.LocalDateTime;
import java.util.List;

public interface BookmarkRepository extends JpaRepository<Bookmark, Long> {

    // Find by title (exact match)
    List<Bookmark> findByTitle(String title);

    // Find by title containing (case-insensitive)
    List<Bookmark> findByTitleContainingIgnoreCase(String title);

    // Find by URL
    List<Bookmark> findByUrl(String url);

    // Find by created date after
    List<Bookmark> findByCreatedAtAfter(LocalDateTime date);

    // Find by created date between
    List<Bookmark> findByCreatedAtBetween(LocalDateTime start, LocalDateTime end);

    // Find by title and URL
    List<Bookmark> findByTitleAndUrl(String title, String url);

    // Find by title or URL
    List<Bookmark> findByTitleContainingOrUrlContaining(String title, String url);

    // Find first 5 by title containing, ordered by created date descending
    List<Bookmark> findTop5ByTitleContainingOrderByCreatedAtDesc(String title);

    // Check if exists by title
    boolean existsByTitle(String title);

    // Count by title containing
    long countByTitleContaining(String title);
}
```

**Common Keywords:**
- `findBy`, `readBy`, `getBy`, `queryBy`
- `countBy`, `existsBy`, `deleteBy`
- `Containing`, `StartingWith`, `EndingWith`
- `IgnoreCase`, `AllIgnoreCase`
- `And`, `Or`
- `GreaterThan`, `LessThan`, `Between`
- `OrderBy...Asc`, `OrderBy...Desc`
- `First`, `Top`, `Distinct`

### Using @Query Annotation

For complex queries, use the `@Query` annotation with JPQL:

```java
package com.example.demo.repository;

import com.example.demo.domain.Bookmark;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Query;
import org.springframework.data.repository.query.Param;

import java.time.LocalDateTime;
import java.util.List;

public interface BookmarkRepository extends JpaRepository<Bookmark, Long> {

    // JPQL query with positional parameter
    @Query("SELECT b FROM Bookmark b WHERE b.title LIKE %?1%")
    List<Bookmark> searchByTitle(String title);

    // JPQL query with named parameter
    @Query("SELECT b FROM Bookmark b WHERE b.title LIKE %:keyword% OR b.url LIKE %:keyword%")
    List<Bookmark> searchByKeyword(@Param("keyword") String keyword);

    // JPQL query with pagination
    @Query("SELECT b FROM Bookmark b WHERE b.createdAt >= :date")
    Page<Bookmark> findRecentBookmarks(@Param("date") LocalDateTime date, Pageable pageable);

    // JPQL query with multiple parameters
    @Query("SELECT b FROM Bookmark b WHERE b.title = :title AND b.createdAt > :date")
    List<Bookmark> findByTitleAndDateAfter(
        @Param("title") String title,
        @Param("date") LocalDateTime date
    );
}
```

### Using Native SQL Queries

For database-specific queries, use native SQL:

```java
package com.example.demo.repository;

import com.example.demo.domain.Bookmark;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Query;
import org.springframework.data.repository.query.Param;

import java.util.List;

public interface BookmarkRepository extends JpaRepository<Bookmark, Long> {

    // Native SQL query
    @Query(value = "SELECT * FROM bookmarks WHERE title ILIKE %:keyword%",
           nativeQuery = true)
    List<Bookmark> searchByTitleNative(@Param("keyword") String keyword);

    // Native SQL with pagination (PostgreSQL specific)
    @Query(value = "SELECT * FROM bookmarks ORDER BY created_at DESC LIMIT :limit OFFSET :offset",
           nativeQuery = true)
    List<Bookmark> findRecentBookmarksNative(@Param("limit") int limit, @Param("offset") int offset);

    // Native SQL for aggregation
    @Query(value = "SELECT COUNT(*) FROM bookmarks WHERE created_at >= CURRENT_DATE",
           nativeQuery = true)
    long countTodayBookmarks();
}
```

**Note:** Use `nativeQuery = true` for native SQL queries.

### Data Modifying Queries with @Modifying

For any data modification such as INSERT/UPDATE/DELETE operations, use `@Modifying` annotation:

```java
package com.example.demo.repository;

import com.example.demo.domain.Bookmark;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Modifying;
import org.springframework.data.jpa.repository.Query;
import org.springframework.data.repository.query.Param;

import java.time.LocalDateTime;

public interface BookmarkRepository extends JpaRepository<Bookmark, Long> {

    // Update query
    @Modifying
    @Query("UPDATE Bookmark b SET b.title = :title WHERE b.id = :id")
    int updateTitle(@Param("id") Long id, @Param("title") String title);

    // Update multiple fields
    @Modifying
    @Query("UPDATE Bookmark b SET b.title = :title, b.url = :url WHERE b.id = :id")
    int updateBookmark(@Param("id") Long id, @Param("title") String title, @Param("url") String url);

    // Delete query
    @Modifying
    @Query("DELETE FROM Bookmark b WHERE b.createdAt < :date")
    int deleteOldBookmarks(@Param("date") LocalDateTime date);

    // Native update query
    @Modifying
    @Query(value = "UPDATE bookmarks SET title = UPPER(title) WHERE id = :id",
           nativeQuery = true)
    int uppercaseTitle(@Param("id") Long id);
}
```

## Spring Data JPA Projections

Projections allow you to fetch only specific fields instead of entire entities, improving performance.

### Interface-based Projections

Create an interface with getter methods for the fields you want:

```java
package com.example.demo.projection;

import java.time.LocalDateTime;

public interface BookmarkSummary {
    Long getId();
    String getTitle();
    Instant getCreatedAt();
    // URL is excluded
}
```

Use the projection in your repository:

```java
package com.example.demo.repository;

import com.example.demo.domain.Bookmark;
import com.example.demo.projection.BookmarkSummary;
import org.springframework.data.jpa.repository.JpaRepository;

import java.util.List;

public interface BookmarkRepository extends JpaRepository<Bookmark, Long> {

    // Returns only id, title, and createdAt
    List<BookmarkSummary> findAllBy();

    List<BookmarkSummary> findByTitleContaining(String title);
}
```

### Class-based Projections (DTOs)

Create a DTO class:

```java
package com.example.demo.dto;

public class BookmarkDTO {
    private Long id;
    private String title;

    public BookmarkDTO(Long id, String title) {
        this.id = id;
        this.title = title;
    }

    // Getters and setters
}
```

Use the DTO in a custom query:

```java
package com.example.demo.repository;

import com.example.demo.domain.Bookmark;
import com.example.demo.dto.BookmarkDTO;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Query;

import java.util.List;

public interface BookmarkRepository extends JpaRepository<Bookmark, Long> {

    @Query("SELECT new com.example.demo.dto.BookmarkDTO(b.id, b.title) FROM Bookmark b")
    List<BookmarkDTO> findAllBookmarkDTOs();
}
```

### Dynamic Projections

Use generic type parameter to choose a projection at runtime:

```java
package com.example.demo.repository;

import com.example.demo.domain.Bookmark;
import org.springframework.data.jpa.repository.JpaRepository;

import java.util.List;

public interface BookmarkRepository extends JpaRepository<Bookmark, Long> {

    <T> List<T> findByTitleContaining(String title, Class<T> type);
}
```

Use it in your service:

```java
// Get full entities
List<Bookmark> fullBookmarks = bookmarkRepository.findByTitleContaining("Java", Bookmark.class);

// Get projections
List<BookmarkSummary> summaries = bookmarkRepository.findByTitleContaining("Java", BookmarkSummary.class);
```

## Spring Data JPA Auditing

Auditing automatically tracks who created/modified an entity and when.

### Enable Auditing

Add `@EnableJpaAuditing` to your main application class or configuration class:

```java
package com.example.demo.config;

import org.springframework.data.jpa.repository.config.EnableJpaAuditing;

@Configuration
@EnableJpaAuditing
public class JpaConfig {
}
```

### Update the Entity

Add auditing fields to your entity:

```java
package com.example.demo.domain;

import jakarta.persistence.*;
import org.springframework.data.annotation.CreatedBy;
import org.springframework.data.annotation.CreatedDate;
import org.springframework.data.annotation.LastModifiedBy;
import org.springframework.data.annotation.LastModifiedDate;
import org.springframework.data.jpa.domain.support.AuditingEntityListener;

import java.time.Instant;

@Entity
@Table(name = "bookmarks")
@EntityListeners(AuditingEntityListener.class)
public class Bookmark {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false)
    private String title;

    @Column(nullable = false)
    private String url;

    @CreatedDate
    @Column(name = "created_at", nullable = false, updatable = false)
    private Instant createdAt;

    @LastModifiedDate
    @Column(name = "updated_at")
    private Instant updatedAt;

    @CreatedBy
    @Column(name = "created_by", updatable = false)
    private String createdBy;

    @LastModifiedBy
    @Column(name = "updated_by")
    private String updatedBy;

    // Constructors, getters, and setters...
}
```

**Key Annotations:**
- `@EntityListeners(AuditingEntityListener.class)` - Enables auditing for this entity
- `@CreatedDate` - Automatically set when entity is created
- `@LastModifiedDate` - Automatically updated when entity is modified
- `@CreatedBy` - User who created the entity
- `@LastModifiedBy` - User who last modified the entity

### Create Migration for Audit Fields

Create a new Flyway migration `V3__add_audit_columns.sql`:

```sql
ALTER TABLE bookmarks ADD COLUMN updated_at TIMESTAMP;
ALTER TABLE bookmarks ADD COLUMN created_by VARCHAR(255);
ALTER TABLE bookmarks ADD COLUMN updated_by VARCHAR(255);
```

### Provide AuditorAware Implementation

To track the user, implement `AuditorAware`:

```java
package com.example.demo.config;

import org.springframework.data.domain.AuditorAware;
import org.springframework.stereotype.Component;

import java.util.Optional;

@Component
public class AuditorAwareImpl implements AuditorAware<String> {

    @Override
    public Optional<String> getCurrentAuditor() {
        // In a real application, get the current user from SecurityContext
        // For now, return a hardcoded value
        return Optional.of("system");

        // With Spring Security, you would do:
        // return Optional.ofNullable(SecurityContextHolder.getContext())
        //     .map(SecurityContext::getAuthentication)
        //     .filter(Authentication::isAuthenticated)
        //     .map(Authentication::getName);
    }
}
```

Now, whenever you save a bookmark, the audit fields will be automatically populated:

```java
Bookmark bookmark = new Bookmark("Spring Boot", "https://spring.io");
bookmarkRepository.save(bookmark);
// createdAt, updatedAt, createdBy, and updatedBy are automatically set
```

## Summary

Spring Data JPA simplifies database operations significantly:

1. **No boilerplate code**: Extend `JpaRepository` to get CRUD operations automatically
2. **Query methods**: Create queries just by defining method names
3. **Custom queries**: Use `@Query` for complex JPQL or native SQL
4. **Pagination and sorting**: Built-in support with `Pageable` and `Sort`
5. **Projections**: Fetch only the fields you need
6. **Auditing**: Automatically track creation and modification metadata

With Spring Data JPA, you can focus on business logic instead of writing repetitive data access code!

## Assignment
* Implement database interactions in jblogger application using Spring Data JPA
* Create a Flyway migration for audit fields
* Enable auditing in your application
* Provide an implementation of `AuditorAware`
* Verify that audit fields are automatically populated when saving or updating entities
* Add pagination to the find all bookmarks API endpoint

---
[Home](../README.md) | [← Previous: Flyway Database Migrations](70-flyway-database-migrations.md) | [Next: Spring Security Basics →](90-spring-security-basics.md)
