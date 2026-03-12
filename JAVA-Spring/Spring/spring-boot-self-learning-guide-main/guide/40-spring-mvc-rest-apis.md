# Spring MVC REST APIs

## Table of Contents
- [What is REST?](#what-is-rest)
- [Importance of HTTP Methods, RESTful URLs, and Status Codes](#importance-of-http-methods-restful-urls-and-status-codes)
- [REST API Examples: Blog Application](#rest-api-examples-blog-application)
- [Writing HTTP Request Handlers with Spring MVC](#writing-http-request-handlers-with-spring-mvc)
- [Mapping HTTP Methods](#mapping-http-methods)
- [Spring MVC Annotations](#spring-mvc-annotations)
- [Using ResponseEntity](#using-responseentity)
- [Validation](#validation)
- [Error Handling](#error-handling)
- [Assignment](#assignment)

## What is REST?

REST (Representational State Transfer) is an architectural style for designing networked applications. 
It relies on stateless, client-server communication using standard HTTP methods. 
In REST, resources are identified by URLs, and operations are performed using HTTP methods like GET, POST, PUT, and DELETE.

Key principles of REST:
- **Client-Server Architecture**: Separation of concerns between client and server
- **Stateless**: Each request contains all information needed to process it
- **Cacheable**: Responses can be cached to improve performance
- **Uniform Interface**: Consistent way to interact with resources using standard HTTP methods

## Importance of HTTP Methods, RESTful URLs, and Status Codes

### HTTP Methods
Using proper HTTP methods provides semantic meaning to API operations:
- **GET**: Retrieve resources (safe and idempotent)
- **POST**: Create new resources
- **PUT**: Update existing resources (idempotent)
- **PATCH**: Partial update of resources
- **DELETE**: Remove resources (idempotent)

### RESTful URLs
Well-designed URLs make APIs intuitive and self-documenting:
- Use nouns for resources, not verbs
- Use plural names for collections
- Use hierarchical structure for relationships
- Keep URLs clean and predictable

### HTTP Status Codes
Status codes communicate the result of operations clearly:
- **2xx**: Success (200 OK, 201 Created, 204 No Content)
- **3xx**: Redirection
- **4xx**: Client errors (400 Bad Request, 404 Not Found, 401 Unauthorized)
- **5xx**: Server errors (500 Internal Server Error)

## REST API Examples: Blog Application

Here are examples of RESTful endpoints for a blog application:

### Resource: Posts
```
GET    /api/posts              - Get all posts
GET    /api/posts/{id}         - Get a specific post
POST   /api/posts              - Create a new post
PUT    /api/posts/{id}         - Update a post
DELETE /api/posts/{id}         - Delete a post
```

### Sub-resource: Comments (belonging to Posts)
```
GET    /api/posts/{postId}/comments              - Get all comments for a post
GET    /api/posts/{postId}/comments/{commentId}  - Get a specific comment
POST   /api/posts/{postId}/comments              - Create a comment on a post
PUT    /api/posts/{postId}/comments/{commentId}  - Update a comment
DELETE /api/posts/{postId}/comments/{commentId}  - Delete a comment
```

### Resource: Authors
```
GET    /api/authors            - Get all authors
GET    /api/authors/{id}       - Get a specific author
GET    /api/authors/{id}/posts - Get all posts by an author
```

**NOTE:** Don't use URLs like `/api/getAllPosts` or `/api/createPost` etc.

## Writing HTTP Request Handlers with Spring MVC
To build REST APIs with Spring MVC, add the following Spring Boot starter dependency:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-webmvc</artifactId>
</dependency>
```

Spring MVC provides annotations to handle HTTP requests in a clean, declarative way.

### @Controller vs @RestController

**@Controller**: Used for traditional MVC applications that return views (HTML pages). 
Methods typically return view names.

```java
@Controller
public class WebController {
    @GetMapping("/home")
    public String home(Model model) {
        return "home"; // Returns view name
    }
}
```

**@RestController**: A convenience annotation that combines `@Controller` and `@ResponseBody`. 
Used for REST APIs that return data (JSON/XML). All methods automatically serialize return values to the response body.

```java
@RestController
public class ApiController {
    @GetMapping("/api/posts")
    public List<Post> getPosts() {
        return postService.getAllPosts(); // Returns JSON
    }
}
```

## Mapping HTTP Methods

Spring provides specialized annotations for different HTTP methods:

### @GetMapping
Used to retrieve resources:

```java
@RestController
@RequestMapping("/api/posts")
public class PostController {

    @GetMapping
    public List<Post> getAllPosts() {
        return postService.getAllPosts();
    }

    @GetMapping("/{id}")
    public Post getPost(@PathVariable Long id) {
        return postService.getPostById(id);
    }
}
```

### @PostMapping
Used to create new resources:

```java
@PostMapping
@ResponseStatus(HttpStatus.CREATED)
public Post createPost(@RequestBody Post post) {
    return postService.createPost(post);
}
```

### @PutMapping
Used to update existing resources:

```java
@PutMapping("/{id}")
public Post updatePost(@PathVariable Long id, @RequestBody Post post) {
    return postService.updatePost(id, post);
}
```

### @DeleteMapping
Used to delete resources:

```java
@DeleteMapping("/{id}")
@ResponseStatus(HttpStatus.NO_CONTENT)
public void deletePost(@PathVariable Long id) {
    postService.deletePost(id);
}
```

### @PatchMapping
Used for partial updates:

```java
@PatchMapping("/{id}")
public Post patchPost(@PathVariable Long id, @RequestBody Map<String, Object> updates) {
    return postService.partialUpdate(id, updates);
}
```

## Spring MVC Annotations

### @PathVariable
Extracts values from the URL path:

```java
@GetMapping("/posts/{postId}/comments/{commentId}")
public Comment getComment(
    @PathVariable Long postId,
    @PathVariable Long commentId
) {
    return commentService.getComment(postId, commentId);
}

// Using custom name
@GetMapping("/authors/{authorId}")
public Author getAuthor(@PathVariable("authorId") Long id) {
    return authorService.getAuthorById(id);
}
```

### @RequestParam
Extracts query parameters from the URL:

```java
// GET /api/posts?page=2&size=10&sort=title
@GetMapping("/posts")
public List<Post> getPosts(
    @RequestParam(defaultValue = "0") int page,
    @RequestParam(defaultValue = "20") int size,
    @RequestParam(required = false) String sort
) {
    return postService.getPosts(page, size, sort);
}
```

### @RequestBody
Binds the HTTP request body to a method parameter (deserializes JSON to Java object):

```java
@PostMapping("/posts")
public Post createPost(@RequestBody Post post) {
    // Spring automatically converts JSON to Post object
    return postService.createPost(post);
}
```

### @ResponseStatus
Sets the HTTP status code for the response:

```java
@PostMapping("/posts")
@ResponseStatus(HttpStatus.CREATED) // Returns 201
public Post createPost(@RequestBody Post post) {
    return postService.createPost(post);
}

@DeleteMapping("/posts/{id}")
@ResponseStatus(HttpStatus.NO_CONTENT) // Returns 204
public void deletePost(@PathVariable Long id) {
    postService.deletePost(id);
}
```

## Using ResponseEntity

`ResponseEntity` provides full control over the HTTP response, including status codes, headers, and body. 
It's more flexible than `@ResponseStatus`.

### Basic Usage

```java
@GetMapping("/{id}")
public ResponseEntity<Post> getPost(@PathVariable Long id) {
    Post post = postService.getPostById(id);
    if (post == null) {
        return ResponseEntity.notFound().build(); // 404
    }
    return ResponseEntity.ok(post); // 200
}
```

### Creating Resources with Location Header

```java
@PostMapping
public ResponseEntity<Post> createPost(@RequestBody Post post) {
    Post created = postService.createPost(post);

    URI location = ServletUriComponentsBuilder
        .fromCurrentRequest()
        .path("/{id}")
        .buildAndExpand(created.getId())
        .toUri();

    return ResponseEntity.created(location).body(created); // 201 with Location header
}
```

### Custom Headers

```java
@GetMapping("/{id}")
public ResponseEntity<Post> getPost(@PathVariable Long id) {
    Post post = postService.getPostById(id);

    return ResponseEntity.ok()
        .header("X-Custom-Header", "CustomValue")
        .header("X-Post-Author", post.getAuthor())
        .body(post);
}
```

### Different Status Codes

```java
@PutMapping("/{id}")
public ResponseEntity<Post> updatePost(
    @PathVariable Long id,
    @RequestBody Post post
) {
    try {
        Post updated = postService.updatePost(id, post);
        return ResponseEntity.ok(updated); // 200
    } catch (PostNotFoundException e) {
        return ResponseEntity.notFound().build(); // 404
    }
}

@DeleteMapping("/{id}")
public ResponseEntity<Void> deletePost(@PathVariable Long id) {
    boolean deleted = postService.deletePost(id);
    if (deleted) {
        return ResponseEntity.noContent().build(); // 204
    }
    return ResponseEntity.notFound().build(); // 404
}
```

### Conditional Responses

```java
@GetMapping("/posts")
public ResponseEntity<List<Post>> getPosts(@RequestParam(required = false) String author) {
    List<Post> posts = postService.getPostsByAuthor(author);

    if (posts.isEmpty()) {
        return ResponseEntity.noContent().build(); // 204
    }

    return ResponseEntity.ok()
        .cacheControl(CacheControl.maxAge(60, TimeUnit.SECONDS))
        .body(posts);
}
```

## Validation
Jakarta Validation is a framework for validating Java beans.
To use Jakarta Validation, add the following Spring Boot starter dependency:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```

Now add the necessary annotations to the request body POJO:

```java
class CreatePostRequest {
    @NotBlank(message = "Title is required")
    @Size(min = 5, max = 100, message = "Title must be between 5 and 100 characters")
    private String title;
    
    @NotBlank(message = "Content is required")
    private String content;
}
```

Now we can use `@Valid` annotation to validate the request body:

```java
@PostMapping
public Post createPost(@Valid @RequestBody CreatePostRequest request) {
    //...
}
```

## Error Handling
If a request handling method in a controller throws an Exception, 
then Spring Boot will handle it and return the response using its default Exception Handling mechanism.

We can handle exceptions in different ways, and depending on your use-case, you can choose one of the approaches that fit best for you.

- Using Controller level @ExceptionHandler
- GlobalExceptionHandler using @RestControllerAdvice
- Spring's ProblemDetails for HTTP APIs (RFC 9457).

### Using Controller level @ExceptionHandler

```java
@RestController
@RequestMapping("/api/posts")
class PostController {

    //...
    //...

    @PostMapping
    ResponseEntity<Post> create(@RequestBody @Valid CreatePostRequest request) {
        //logic that might throw PostSlugAlreadyExistsException
    }

    @PutMapping("/{id}")
    ResponseEntity<Post> update(@PathVariable Long id, @RequestBody @Valid UpdatePostRequest request) {
        //logic that might throw PostSlugAlreadyExistsException
    }
    
    @ExceptionHandler(PostSlugAlreadyExistsException.class)
    public ResponseEntity<ApiError> handle(PostSlugAlreadyExistsException e) {
        ApiError error = new ApiError(e.getMessage());
        return ResponseEntity.status(HttpStatus.BAD_REQUEST).body(error);
    }
}
```

In this approach, you don't have to duplicate the exception handling logic in multiple handler methods in the controller. 
If `PostSlugAlreadyExistsException` is thrown from `create(…)` or `update(…)` methods, 
they will be handled by the respective `@ExceptionHandler` methods.

### GlobalExceptionHandler using @RestControllerAdvice
What if the same type of exceptions may occur in different Controllers, and we want to handle those Exceptions in the same way? 
In such cases, we can use the Global Exception Handling approach by using `@RestControllerAdvice`.

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(PostNotFoundException.class)
    public ResponseEntity<ApiError> handle(PostNotFoundException e) {
        ApiError error = new ApiError(e.getMessage());
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(error);
    }

    @ExceptionHandler(PostSlugAlreadyExistsException.class)
    public ResponseEntity<ApiError> handle(PostSlugAlreadyExistsException e) {
        ApiError error = new ApiError(e.getMessage());
        return ResponseEntity.status(HttpStatus.BAD_REQUEST).body(error);
    }
}
```

By using ControllerAdvice approach, we don't have to duplicate the same `@ExceptionHandler` logic in multiple Controllers.

**IMPORTANT:** If you have an `@ExceptionHandler` handling the same Exception in both Controller 
and `GlobalExceptionHandler` then Controller level `@ExceptionHandler` method takes priority.

### Using ProblemDetails API
Spring Framework 6 implemented the Problem Details for HTTP APIs specification, RFC 9457.

```java
@RestControllerAdvice
public class GlobalExceptionHandler extends ResponseEntityExceptionHandler {
    
    @ExceptionHandler(PostNotFoundException.class)
    ProblemDetail handle(PostNotFoundException e) {
        ProblemDetail problemDetail = ProblemDetail.forStatusAndDetail(HttpStatus.NOT_FOUND, e.getMessage());
        problemDetail.setTitle("Post Not Found");
        problemDetail.setType(URI.create("https://api.jblogger.com/errors/not-found"));
        return problemDetail;
    }

    @ExceptionHandler(PostSlugAlreadyExistsException.class)
    ProblemDetail handle(PostSlugAlreadyExistsException e) {
        ProblemDetail problemDetail = ProblemDetail.forStatusAndDetail(HttpStatus.BAD_REQUEST, e.getMessage());
        problemDetail.setTitle("Post Slug Already Exists");
        problemDetail.setType(URI.create("https://api.jblogger.com/errors/bad-request"));
        return problemDetail;
    }
}
```

Now when you make the API call to fetch post with non-existing id then you will get the following response:

```json
{
  "type": "https://api.jblogger.com/errors/not-found",
  "title": "Post Not Found",
  "status": 404,
  "detail": "Post with id: 111 not found",
  "instance": "/api/posts/111"
}
```

In addition to the standard fields type, title, status, detail, instance we can also include custom properties as follows:

```java
@RestControllerAdvice
public class GlobalExceptionHandler extends ResponseEntityExceptionHandler {

    @ExceptionHandler(PostNotFoundException.class)
    ProblemDetail handle(PostNotFoundException e) {
        ProblemDetail problemDetail = ProblemDetail.forStatusAndDetail(HttpStatus.NOT_FOUND, e.getMessage());
        problemDetail.setTitle("Post Not Found");
        problemDetail.setType(URI.create("https://api.jblogger.com/errors/not-found"));
        problemDetail.setProperty("errorCategory", "Generic");
        problemDetail.setProperty("timestamp", Instant.now());
        return problemDetail;
    }
}
```

**NOTE:** It is a good practice to always handle all types of exceptions and return a standard response using Problem Details API.

## Assignment
Implement the following REST API endpoints for the Blog application using Spring MVC.

- Get all posts
- Get a post by id
- Create a new post
- Update a post
- Delete a post

### Post
Use the following Post class for the data model:

```java
class Post {
    private Long id;
    private String slug;
    private String title;
    private String content;
    
    // Getters and setters
}
```

### PostRepository
Use the following PostRepository using InMemory HashMap as data store:

```java
class PostRepository {
    private static final AtomicLong counter = new AtomicLong();
    private Map<Long, Post> posts = new HashMap<>();
    
    public List<Post> getAllPosts() {
        return new ArrayList<>(posts.values());
    }
    
    public Post getPostById(Long id) {
        return posts.get(id);
    }
    
    public Post createPost(Post post) {
        post.setId(counter.incrementAndGet());
        posts.put(post.getId(), post);
        return post;
    }
    
    public Post updatePost(Long id, Post post) {
        post.setId(id);
        posts.put(id, post);
    }
    
    public boolean deletePost(Long id) {
        return posts.remove(id) != null;
    }
}
```

---
[Home](../README.md) | [← Previous: Core Concepts](30-core-concepts.md) | [Next: Spring JdbcClient →](50-jdbc-client.md)
