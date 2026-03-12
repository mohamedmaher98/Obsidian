# Getting Started

## Table of Contents
- [Prerequisites](#prerequisites)
- [Running a Postgres Database](#running-a-postgres-database)
- [Creating a Spring Boot Project](#creating-a-spring-boot-project)

## Prerequisites
To follow along with this guide, you will need to install the following:

- JDK 21 or later (prefer JDK 25)
- [IntelliJ IDEA](https://www.jetbrains.com/idea/)
- [Docker](https://www.docker.com/) & Docker Compose
- [Git](https://git-scm.com/)

I would recommend using:

- [SDKMAN](https://sdkman.io/) to install JDK
- [JetBrains Toolbox](https://www.jetbrains.com/toolbox-app/) to install IntelliJ IDEA
- Docker Desktop for Mac/Windows

## Running a Postgres Database
Once you have installed these, you can start a Postgres database using Docker Compose.

Create a file called `docker-compose.yml` with the following contents:

```yaml
services:
  postgres:
    image: postgres:18
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
      POSTGRES_DB: postgres
    ports:
      - "5432:5432"
```

Run `docker compose up -d` to start the database.

## Creating a Spring Boot Project
You can create a new Spring Boot project using IntelliJ IDEA or by going to https://start.spring.io/.

Generate a new Spring Boot project by using the following details:

```shell
Language: Java
Build tool: Maven
Spring Boot version: 4.0.1
Group: com.sivalabs
Artifact: jblogger
Name: jblogger
Package name: com.sivalabs.jblogger
Configuration: Properties
Java version: 25
Dependencies: Web
```

Generate the project, unzip and import it into IntelliJ IDEA.

---
[Home](../README.md) | [← Previous: Introduction](10-introduction.md) | [Next: Core Concepts →](30-core-concepts.md)
