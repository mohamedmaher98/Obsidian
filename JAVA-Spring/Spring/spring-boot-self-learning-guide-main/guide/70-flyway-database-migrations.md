# Flyway Database Migrations

## Table of Contents
- [Create Flyway Migration Scripts](#create-flyway-migration-scripts)
- [Customizing Flyway Configuration](#customizing-flyway-configuration)
- [Java based Flyway Migrations](#java-based-flyway-migrations)
- [Assignment](#assignment)

In the JdbcClient section, we have seen how to initialize the database using **schema.sql** and **data.sql** scripts.
This may be useful for demos and quick prototypes, but for real-world applications we should use a database migration tool.

[Flyway](https://flywaydb.org/) is one of the most popular Java-based database migration libraries.
Spring Boot provides out-of-the-box support for Flyway database migrations.

**Why should we use a database migration tool?**

- Versioned, immutable migrations
- Each migration runs exactly once
- Schema history stored in the database
- Safe locking and fail-fast behavior
- CI/CD and production-ready by design

Let us see how we can use Flyway for implementing database migrations in a Spring Boot application that uses PostgreSQL database.

Add the following dependencies to `pom.xml`:

```xml
<dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-flyway</artifactId>
</dependency>
<dependency>
    <groupId>org.flywaydb</groupId>
    <artifactId>flyway-database-postgresql</artifactId>
</dependency>
```

## Create Flyway Migration Scripts
Flyway follows the `V<VERSION>__<DESCRIPTION>.sql` (Capital V, version number, 2 underscores, description) naming convention for its versioned migration scripts.

Let's add the following two migration scripts under **src/main/resources/db/migration** folder.

**V1__create_tables.sql**
```sql
create table bookmarks
(
    id         bigserial not null,
    title      varchar   not null,
    url        varchar   not null,
    created_at timestamp,
    primary key (id)
);
```

**V2__create_bookmarks_indexes.sql**
```sql
CREATE INDEX idx_bookmarks_title ON bookmarks(title);
```

Now when you start the application, you should notice the following Flyway execution related logs as follows:

```shell
INFO 4009 --- [           main] o.f.c.i.database.base.BaseDatabaseType   : Database: jdbc:postgresql://localhost:5432/postgres (PostgreSQL 18)
INFO 4009 --- [           main] o.f.c.i.s.JdbcTableSchemaHistory         : Schema history table "public"."flyway_schema_history" does not exist yet
INFO 4009 --- [           main] o.f.core.internal.command.DbValidate     : Successfully validated 2 migrations (execution time 00:00.010s)
INFO 4009 --- [           main] o.f.c.i.s.JdbcTableSchemaHistory         : Creating Schema History table "public"."flyway_schema_history" ...
INFO 4009 --- [           main] o.f.core.internal.command.DbMigrate      : Current version of schema "public": << Empty Schema >>
INFO 4009 --- [           main] o.f.core.internal.command.DbMigrate      : Migrating schema "public" to version "1 - create tables"
INFO 4009 --- [           main] o.f.core.internal.command.DbMigrate      : Migrating schema "public" to version "2 - create bookmarks indexes"
INFO 4009 --- [           main] o.f.core.internal.command.DbMigrate      : Successfully applied 2 migrations to schema "public", now at version v2 (execution time 00:00.041s)
```

Flyway keeps track of all the applied migrations history in **flyway_schema_history** table by default.
If you check the data in **flyway_schema_history** table now, you can see the following rows:

```shell
| installed_rank | version | description              | type | script                           | checksum   | installed_by | installed_on               | execution_time | success |
|:---------------|:--------|:-------------------------|:-----|:---------------------------------|:-----------|:-------------|:---------------------------|:---------------|:--------|
| 1              | 1       | create tables            | SQL  | V1__create_tables.sql            | 1020037327 | test         | 2026-01-06 09:13:04.439012 | 6              | true    |
| 2              | 2       | create bookmarks indexes | SQL  | V2__create_bookmarks_indexes.sql | 732086927  | test         | 2026-01-06 09:13:04.456876 | 4              | true    |
```

If you keep the same database instance running and restart the application, then Flyway doesn't re-run the already applied migrations.
If you have added any new migrations, then only those migration scripts will be executed.


**Important Flyway Rules:**

The following rules must be followed, otherwise Flyway will throw error while applying migrations:

* There should be no duplicate version numbers in flyway migration script file names.

  Ex: **V1__init.sql**, **V1__indexes.sql**. Here version number 1 is used multiple times.
* Once a migration is applied, you should not change its content.


## Customizing Flyway Configuration
Spring Boot provides sensible defaults for Flyway migrations,
but you can configure various Flyway configuration properties
using **spring.flyway.{property-name}** properties in **application.properties** file.

### Flyway migrations for different databases
If you are building a product which can be used with different databases, then you can configure flyway migration locations as follows:

```properties
spring.flyway.locations=classpath:db/migration/{vendor}
```

Then you can place **mysql** specific scripts under **src/main/resources/db/migration/mysql** directory,
**postgresql** specific scripts under **src/main/resources/db/migration/postgresql**, etc.
You can check the vendor names in **org.springframework.boot.jdbc.DatabaseDriver** class.

In addition to this, some interesting flyway configuration properties are:

```properties
# disable flyway execution
spring.flyway.enabled=false

# If you have an existing database and start using Flyway for new database changes.
spring.flyway.baseline-on-migrate=true

# To customize flyway migrations tracking table name
spring.flyway.table=db_migrations
```

## Java based Flyway Migrations
In addition to SQL scripts, you can also write database migrations using Java classes.

You can create **V3__InsertSampleData.java** in **db.migration** package as follows:

```java
package db.migration;

import org.flywaydb.core.api.migration.BaseJavaMigration;
import org.flywaydb.core.api.migration.Context;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.jdbc.datasource.SingleConnectionDataSource;

public class V3__InsertSampleData extends BaseJavaMigration {

  public void migrate(Context context) {
      JdbcClient jdbcClient = JdbcClient.create(
              new SingleConnectionDataSource(context.getConnection(), true));

      jdbcClient.sql("...").update();
  }
}
```

If the database changes involve complex logic which is difficult to write using plain SQL,
then Java-based migrations come very handy.

You can use Flyway Database Migrations with different persistence technologies
such as **JdbcClient**, **Spring Data Jdbc**, **Spring Data JPA**, **jOOQ**, etc.

## Assignment
- Remove the **schema.sql** and **data.sql** scripts from the application.
- Use Flyway database migrations to create **users**, **posts**, **comments** tables and insert sample data.

---
[Home](../README.md) | [← Previous: Database Transaction Management](60-db-transaction-management.md) | [Next: Spring Data JPA →](80-spring-data-jpa.md)
