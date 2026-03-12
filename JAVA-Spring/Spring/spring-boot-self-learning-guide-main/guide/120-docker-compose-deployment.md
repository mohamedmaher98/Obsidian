# Docker Compose Deployment

## Table of Contents
- [Introduction](#introduction)
- [Dockerizing a Spring Boot Application](#dockerizing-a-spring-boot-application)
- [Using Paketo Buildpacks](#using-paketo-buildpacks)
- [Deploying with Docker Compose](#deploying-with-docker-compose)
- [Assignments](#assignments)

## Introduction

Docker enables you to package your Spring Boot application with all its dependencies into a standardized container 
that can run consistently across different environments. Docker Compose simplifies the orchestration of multi-container applications, 
making it ideal for deploying Spring Boot applications with databases and other services.

In this guide, you'll learn multiple approaches to containerize your Spring Boot application and 
deploy it alongside a PostgreSQL database using Docker Compose.

## Dockerizing a Spring Boot Application

### Using Dockerfile

The simplest approach is to create a Dockerfile that copies your JAR file and runs it.

#### Basic Dockerfile

```dockerfile
# Use an official OpenJDK runtime as base image
FROM eclipse-temurin:25-jre

# Set the working directory
WORKDIR /app

# Copy the JAR file into the container
COPY target/*.jar app.jar

# Expose the application port
EXPOSE 8080

# Run the application
ENTRYPOINT ["java", "-jar", "app.jar"]
```

#### Building and Running

```bash
# Build your Spring Boot application
./mvnw clean package

# Build the Docker image
docker build -t my-spring-app:1.0 .

# Run the container
docker run -p 8080:8080 my-spring-app:1.0
```

### Using Multistage Dockerfile with Layer Caching

Multistage builds optimize your Docker images by separating the build environment from the runtime environment. 
They also enable better layer caching, reducing build times.

#### Multistage Dockerfile

```dockerfile
# Stage 1: Build stage
FROM eclipse-temurin:25-jdk-alpine AS builder

WORKDIR /workspace/app

# Copy Maven wrapper and pom.xml first (for dependency caching)
COPY mvnw .
COPY .mvn .mvn
COPY pom.xml .

# Download dependencies (this layer will be cached)
RUN ./mvnw dependency:go-offline -B

# Copy source code
COPY src src

# Build the application
RUN ./mvnw package -DskipTests

# Extract layers for better caching
RUN java -Djarmode=layertools -jar target/*.jar extract

# Stage 2: Runtime stage
FROM eclipse-temurin:25-jre

WORKDIR /app

# Copy extracted layers from builder stage
COPY --from=builder /app/dependencies/ ./
COPY --from=builder /app/spring-boot-loader/ ./
COPY --from=builder /app/snapshot-dependencies/ ./
COPY --from=builder /app/application/ ./
ENTRYPOINT ["java", "org.springframework.boot.loader.launch.JarLauncher"]
```

#### Benefits of Layered Approach

1. **Faster Builds**: Dependencies are cached separately from application code
2. **Smaller Images**: Only necessary runtime components are included
3. **Optimal Caching**: Each layer is cached independently

#### Building with Multistage Dockerfile

```bash
# Build the image (no need to run mvnw separately)
docker build -t my-spring-app:1.0 .

# The build happens inside Docker, ensuring consistency
```

### Using Paketo Buildpacks

Paketo Buildpacks provide a modern, cloud-native approach to building container images without writing Dockerfiles. 
They automatically detect your application type and apply best practices.

#### What are Buildpacks?

Buildpacks are tools that transform your application source code into container images without requiring a Dockerfile. They:
- Automatically detect application type
- Install required dependencies
- Apply security patches
- Optimize for production
- Follow best practices

#### Using Spring Boot Maven Plugin with Buildpacks

The Spring Boot Maven plugin includes built-in support for Buildpacks.

```xml
<plugin>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-maven-plugin</artifactId>
</plugin>
```

Now we can build the image using the `spring-boot:build-image` goal:

```bash
# Build image using Buildpacks
./mvnw spring-boot:build-image

# The image name defaults to: docker.io/library/${project.artifactId}:${project.version}
```

#### Customizing Buildpack Configuration in pom.xml

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <configuration>
                <image>
                    <name>my-registry.com/${project.artifactId}:${project.version}</name>
                    <env>
                        <BP_JVM_VERSION>25</BP_JVM_VERSION>
                        <BPE_JAVA_TOOL_OPTIONS>-XX:MaxRAMPercentage=75.0</BPE_JAVA_TOOL_OPTIONS>
                    </env>
                    <buildpacks>
                        <buildpack>paketo-buildpacks/java</buildpack>
                    </buildpacks>
                </image>
            </configuration>
        </plugin>
    </plugins>
</build>
```

#### Running the Buildpack-Created Image

```bash
# Run the image
docker run -p 8080:8080 my-spring-app:1.0
```

#### Benefits of Paketo Buildpacks

1. **No Dockerfile Required**: Automated image creation
2. **Security**: Regular security updates and patches
3. **Optimization**: Automatic layer optimization and caching
4. **Consistency**: Standard images across teams
5. **Best Practices**: Built-in production-ready configurations

## Deploying with Docker Compose

Docker Compose allows you to define and run multi-container applications. 
It's perfect for deploying Spring Boot applications with databases and other services.

### Creating docker-compose.yml

Create a `docker-compose.yml` file in your project root:

```yaml
services:
  # PostgreSQL Database
  postgres:
    image: postgres:18-alpine
    environment:
      POSTGRES_DB: myappdb
      POSTGRES_USER: dbuser
      POSTGRES_PASSWORD: dbpassword
    ports:
      - "5432:5432"
    volumes:
      - postgres-data:/var/lib/postgresql/data
      # If you want to run initialization scripts to initialize the database,
      # keep the SQL scripts in init-scripts directory in the same directory as docker-compose.yml
      # - ./init-scripts:/docker-entrypoint-initdb.d
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U dbuser -d myappdb"]
      interval: 10s
      timeout: 5s
      retries: 5

  # Spring Boot Application
  app:
    image: my-spring-app:1.0
    container_name: spring-boot-app
    depends_on:
      postgres:
        condition: service_healthy
    environment:
      SPRING_DATASOURCE_URL: jdbc:postgresql://postgres:5432/myappdb
      SPRING_DATASOURCE_USERNAME: dbuser
      SPRING_DATASOURCE_PASSWORD: dbpassword
    ports:
      - "8080:8080"
    restart: unless-stopped

volumes:
  postgres-data:
    driver: local
```

### Running the Application

#### Start All Services

```bash
# Build and start all services in detached mode
docker compose up -d --build

# View logs
docker compose logs -f

# View logs for specific service
docker compose logs -f app
```

Now you can access the application at http://localhost:8080.

#### Common Docker Compose Commands

```bash
# Stop all services
docker compose stop

# Stop and remove containers, networks
docker compose down

# Stop and remove containers, networks, and volumes
docker compose down -v

# Restart a specific service
docker compose restart app

# View running services
docker compose ps

# Execute command in running container
docker compose exec app bash

# View resource usage
docker compose stats

# Rebuild specific service
docker compose up -d --build app
```


## Assignments

### Assignment 1: Basic Dockerization
**Objective**: Create a basic Dockerfile for your Spring Boot application.

**Tasks**:
1. Create a simple Dockerfile that packages your Spring Boot JAR
2. Build the Docker image
3. Run the container and test the application

### Assignment 2: Multistage Dockerfile
**Objective**: Optimize your Docker image using multistage builds.

**Tasks**:
1. Convert your basic Dockerfile to a multistage build
2. Implement layer caching for dependencies
3. Compare the image sizes before and after

### Assignment 3: Paketo Buildpacks
**Objective**: Build your application using Paketo Buildpacks.

**Tasks**:
1. Configure the Spring Boot Maven plugin
2. Build the image using `./mvnw spring-boot:build-image`
3. Customize environment variables for JVM settings
4. Compare build time and image size with Dockerfile approach
5. Document the differences and benefits

### Assignment 4: Docker Compose Deployment
**Objective**: Deploy your application with PostgreSQL using Docker Compose.

**Tasks**:
1. Create a `docker-compose.yml` with jblogger app and PostgreSQL
2. Configure environment variables for database connection
3. Add a volume for database persistence
4. Test the complete setup

[Home](../README.md) | [← Previous: Testing](110-testing.md) | [Next: Conclusion →](130-conclusion.md)
