# Introduction

- Spring framework was created as an alternative to J2EE (J2EE was renamed to Java EE and then to Jakarta EE)
- Spring provides Dependency Injection, Aspect Oriented Programming, and many other features to simplify Java development.
- Spring provides many modules (Jdbc, ORM, Security, Batch, etc) to build enterprise applications.
- Spring initially provided XML based configuration, but later introduced Java based configuration and annotations.
- However, configuring Spring applications were still challenging and usually involved a lot of boilerplate code copy-pasted from previous projects.
- Spring Boot is created to simplify the process of creating Spring applications.
- Spring Boot provides opinionated defaults, auto-configuration, and starter dependencies to reduce boilerplate code and make development faster.
- Spring Boot also provides embedded servers support to run applications as fat/uber jar.

**I strongly believe that the best way to learn a new technology is to build something with it.**

**Throughout this guide, you will learn how to use Spring Boot by building a Blog REST API (called jblogger) progressively.
In this guide, each section will explain a specific feature of Spring Boot and will give an assignment to build jblogger**

## jblogger
- Admin user can create, update, delete posts.
- Users can register and login.
- Anonymous users can view posts.
- Authenticated users can add comments.
- Admin users can view and delete comments.

Though it is a very simple REST API, it covers most of the Spring Boot features and best practices.

### Database
- We will use the PostgreSQL database for this guide. But you can use any database of your choice.
- We will use Flyway for database migrations.
- We will run PostgreSQL in a Docker container using Docker Compose.

#### USERS Table

| Column     | Type         | Constraints                 |
|------------|--------------|-----------------------------|
| ID         | BIGINT       | PRIMARY KEY, AUTO_INCREMENT |
| FULL_NAME  | VARCHAR(255) | NOT NULL                    |
| EMAIL      | VARCHAR(255) | NOT NULL, UNIQUE            |
| PASSWORD   | VARCHAR(255) | NOT NULL                    |
| ROLE       | VARCHAR(50)  | NOT NULL                    |
| CREATED_AT | TIMESTAMP    | NOT NULL                    |
| UPDATED_AT | TIMESTAMP    |                             |

#### POSTS Table

| Column     | Type         | Constraints                 |
|------------|--------------|-----------------------------|
| ID         | BIGINT       | PRIMARY KEY, AUTO_INCREMENT |
| TITLE      | VARCHAR(255) | NOT NULL                    |
| SLUG       | VARCHAR(255) | NOT NULL, UNIQUE            |
| CONTENT    | TEXT         | NOT NULL                    |
| AUTHOR_ID  | BIGINT       | NOT NULL                    |
| STATUS     | VARCHAR(50)  | NOT NULL                    |
| CREATED_AT | TIMESTAMP    | NOT NULL                    |
| UPDATED_AT | TIMESTAMP    |                             |

#### COMMENTS Table

| Column     | Type         | Constraints                 |
|------------|--------------|-----------------------------|
| ID         | BIGINT       | PRIMARY KEY, AUTO_INCREMENT |
| POST_ID    | BIGINT       | NOT NULL                    |
| AUTHOR_ID  | BIGINT       | NOT NULL                    |
| CONTENT    | TEXT         | NOT NULL                    |
| CREATED_AT | TIMESTAMP    | NOT NULL                    |
| UPDATED_AT | TIMESTAMP    |                             |

### REST API Endpoints

#### Login

- `POST /api/login`: Authenticate a user and return a JWT token.

#### Register

- `POST /api/register`: Register a new user.

#### Create Post

- `POST /api/posts`: Create a new post.

#### Get Posts

- `GET /api/posts?page=1&size=10`: Retrieve a page of posts.

#### Get Post By Slug

- `GET /api/posts/{slug}`: Retrieve a post by its slug.

#### Update Post

- `PUT /api/posts/{slug}`: Update a post by its slug.

#### Delete Post

- `DELETE /api/posts/{slug}`: Delete a post by its slug.

#### Add Comment

- `POST /api/posts/{slug}/comments`: Add a comment to a post.

#### Get Comments By a Post Slug

- `GET /api/posts/{slug}/comments`: Retrieve comments for a post by its slug.

#### Delete Comment

- `DELETE /api/comments/{id}`: Delete a comment by its ID.

---
[Home](../README.md) | [Next: Getting Started →](20-getting-started.md)
