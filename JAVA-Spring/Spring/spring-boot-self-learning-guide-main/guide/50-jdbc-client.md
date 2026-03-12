# Spring JdbcClient

## Table of Contents
- [Overview](#overview)
- [Working with H2 Database](#working-with-h2-database)
- [Database Initialization](#database-initialization)
- [Using Spring JdbcClient](#using-spring-jdbcclient)
- [Adding PostgreSQL Database Support](#adding-postgresql-database-support)
- [Assignment](#assignment)

## Overview

Spring's JDBC support simplifies working with SQL databases by eliminating boilerplate code commonly required with traditional JDBC. 
Instead of manually managing database connections, creating statements, handling result sets, and dealing with exception handling, 
Spring provides higher-level abstractions like `JdbcClient` (introduced in Spring Framework 6.1) 
and `JdbcTemplate` that handle these concerns automatically. 
This allows developers to focus on writing SQL queries and mapping results to domain objects, 
while Spring manages resource management, exception translation, and connection pooling behind the scenes.

## Working with H2 Database

H2 is an embedded in-memory database that's good enough for development and testing. 
Spring Boot automatically configures all necessary beans to interact with H2 when it detects the H2 dependency on the classpath.

### Adding H2 Dependency
Add the JDBC starter and H2 database driver dependency to your `pom.xml`:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <scope>runtime</scope>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-h2console</artifactId>
</dependency>
```

### Auto-Configuration

Spring Boot automatically configures the following beans when JDBC starter and H2 are on the classpath:

- **DataSource**: Connection pool for managing database connections
- **JdbcClient/JdbcTemplate**: For executing SQL queries
- **DataSourceTransactionManager**: For transaction management
- **H2 Console**: Web-based database console (accessible at `/h2-console` when enabled)

### Configuration Properties

Configure H2 in your `application.properties`:

```properties
# Enable H2 Console
spring.h2.console.enabled=true
spring.h2.console.path=/h2-console
```

With this configuration, you can access the H2 console at `http://localhost:8080/h2-console` and connect using the JDBC URL `jdbc:h2:mem:testdb`.

## Database Initialization

Spring Boot provides automatic database initialization using SQL script files. 
This is useful for creating schema and loading initial data during application startup.

### Using schema.sql

Create a `schema.sql` file in `src/main/resources` to define your database schema:

```sql
DROP TABLE IF EXISTS bookmarks;

create table bookmarks
(
    id         bigserial primary key,
    title      varchar   not null,
    url        varchar   not null,
    created_at timestamp default now()
);
```

### Using data.sql

Create a `data.sql` file in `src/main/resources` to populate initial data:

```sql
DELETE FROM bookmarks;

INSERT INTO bookmarks(title, url) VALUES
('How (not) to ask for Technical Help?','https://sivalabs.in/how-to-not-to-ask-for-technical-help'),
('Getting Started with Kubernetes','https://sivalabs.in/getting-started-with-kubernetes'),
('Few Things I learned in the HardWay in 15 years of my career','https://sivalabs.in/few-things-i-learned-the-hardway-in-15-years-of-my-career'),
('All the resources you ever need as a Java & Spring application developer','https://sivalabs.in/all-the-resources-you-ever-need-as-a-java-spring-application-developer'),
('SpringBoot Integration Testing using Testcontainers Starter','https://sivalabs.in/spring-boot-integration-testing-using-testcontainers-starter'),
('Testing SpringBoot Applications','https://sivalabs.in/spring-boot-testing')
;
```

### Configuration

Control database initialization behavior in `application.properties`:

```properties
# Database initialization mode
# always: Always initialize (default for embedded databases)
# never: Never initialize
# embedded: Only initialize embedded databases
spring.sql.init.mode=always
```

For platform-specific initialization, you can create files like `schema-h2.sql`, `schema-postgresql.sql`, `data-h2.sql`, etc.

### Using Spring JdbcClient
Spring framework 6.1 introduced a new **JdbcClient** API, which is a wrapper on top of **JdbcTemplate**, 
for performing database operations using a fluent API.

**NOTE:** **JdbcClient** is recommended over **JdbcTemplate** for new projects.

Let's explore how we can use **JdbcClient** to perform various database operations.

Create a Bookmark domain class.

```java
import java.time.Instant;

public record Bookmark(
        Long id, 
        String title, 
        String url, 
        Instant createdAt
) {}
```

Let's implement CRUD operations on Bookmark domain class using JdbcClient API.

```java
@Repository
public class BookmarkRepository {
    private final JdbcClient jdbcClient;

    public BookmarkRepository(JdbcClient jdbcClient) {
        this.jdbcClient = jdbcClient;
    }
    //...
    // ...
}
```

**Fetch all bookmarks:**

```java
public List<Bookmark> findAll() {
    String sql = "select id, title, url, created_at from bookmarks";
    return jdbcClient.sql(sql).query(new BookmarkRowMapper()).list();
}

static class BookmarkRowMapper implements RowMapper<Bookmark> {
    @Override
    public Bookmark mapRow(ResultSet rs, int rowNum) throws SQLException {
        return new Bookmark(
                rs.getLong("id"),
                rs.getString("title"),
                rs.getString("url"),
                rs.getTimestamp("created_at").toInstant()
        );
    }
}     
```

We can also simplify this as follows:

```java 
public List<Bookmark> findAll() {
    String sql = "select id, title, url, created_at from bookmarks";
    return jdbcClient.sql(sql).query(Bookmark.class).list();
}
```

The **JdbcClient** API will take care of dynamically creating a **RowMapper** by using **SimplePropertyRowMapper**. 
It will perform the mapping between bean property names to table column names by converting camelCase to underscore notation.

**Find bookmark By ID:**

We can fetch a bookmark by id using JdbcClient as follows:

```java
public Optional<Bookmark> findById(Long id) {
    String sql = "select id, title, url, created_at from bookmarks where id = :id";
    return jdbcClient.sql(sql).param("id", id).query(Bookmark.class).optional();
    
    // If you want to use your own RowMapper
    //return jdbcClient.sql(sql).param("id", id).query(new BookmarkRowMapper()).optional();
}
```

**Insert a bookmark:**

We can insert a new row into the bookmarks table and get the generated primary key value as follows:

```java
public Long save(Bookmark bookmark) {
    String sql = "insert into bookmarks(title, url) values(:title,:url)";
    KeyHolder keyHolder = new GeneratedKeyHolder();
    jdbcClient.sql(sql)
                .param("title", bookmark.title())
                .param("url", bookmark.url())
                .update(keyHolder);
    return (Long) keyHolder.getKeys().get("ID");
}
```

**Update a bookmark:**

We can update a bookmark as follows:

```java
public void update(Bookmark bookmark) {
    String sql = "update bookmarks set title = ?, url = ? where id = ?";
    int count = jdbcClient.sql(sql)
            .param(1, bookmark.title())
            .param(2, bookmark.url())
            .param(3, bookmark.id())
            .update();
    if (count == 0) {
        throw new RuntimeException("Bookmark not found");
    }
}
```

**Delete a bookmark:**

We can delete a bookmark as follows:

```java
public void delete(Long id) {
    String sql = "delete from bookmarks where id = ?";
    int count = jdbcClient.sql(sql).param(1, id).update();
    if (count == 0) {
        throw new RuntimeException("Bookmark not found");
    }
}
```

## Adding PostgreSQL Database Support

For production environments, you'll typically want to use a more robust database like PostgreSQL or MySQL.

Let's replace H2 with PostgreSQL in this guide.

### Adding PostgreSQL Dependency

Add the PostgreSQL driver to your `pom.xml`:

```xml
<dependency>
    <groupId>org.postgresql</groupId>
    <artifactId>postgresql</artifactId>
    <scope>runtime</scope>
</dependency>
```

### PostgreSQL Configuration

In the **Getting Started** section, we have started the PostgreSQL server using Docker.

Configure PostgreSQL connection in `application.properties`:

```properties
# PostgreSQL Configuration
spring.datasource.driver-class-name=org.postgresql.Driver
spring.datasource.url=jdbc:postgresql://localhost:5432/postgres
spring.datasource.username=postgres
spring.datasource.password=postgres
```

When we explicitly configure the JDBC DataSource properties, 
Spring Boot will automatically configure DataSource using the provided properties instead of using the H2 in-memory database.

## Assignment

- In the previous section, we have implemented REST API endpoints but stored data in **HashMap**.
- Let's store data in a relational database using Spring **JdbcClient**.
- Create **schema.sql** and **data.sql** scripts to create and initialize **posts** table.
- Update **PostRepository** to use **JdbcClient** and persist data in the database instead of HashMap.

---
[Home](../README.md) | [← Previous: Spring MVC REST APIs](40-spring-mvc-rest-apis.md) | [Next: Database Transaction Management →](60-db-transaction-management.md)
