# Database Transaction Management

## Table of Contents
- [Transaction Management using plain JDBC](#transaction-management-using-plain-jdbc)
- [Declarative Transaction Management using @Transactional annotation](#declarative-transaction-management-using-transactional-annotation)
- [Programmatic Transaction Management using TransactionTemplate](#programmatic-transaction-management-using-transactiontemplate)
- [Assignment](#assignment)

A database transaction is a single unit of work, which either completes fully or does not complete at all and leaves the database in a consistent state. 
While implementing database transactions, you need to take ACID (Atomicity, Consistency, Isolation, Durability) properties into consideration.

Let's understand how we can handle database transactions in our Spring Boot applications.

## Transaction Management using plain JDBC

First, let's take a quick look at how we usually handle database transactions in plain JDBC.

```java
class UserService {
    
    void register(User user) {
        String sql = "...";
        Connection conn = dataSource.getConnection(); // <1>
        try(conn) {  // <6>
            conn.setAutoCommit(false);  // <2>
            PreparedStatement pstmt = conn.prepareStatement(sql);
            pstmt.setString(1, user.getEmail());
            pstmt.setString(2, user.getPassword());
            pstmt.executeUpdate();  // <3>
            conn.commit();  // <4>
        } catch(SQLException e) {
            conn.rollback();  // <5>
        }
    }
}
```

In the above code snippet:

* (1) We got a database connection from the DataSource connection pool
* (2) By default, every database operation will be running in its own transaction automatically. 
      To take control of defining our transaction scope, we set AutoCommit to false.
* (3) Then we performed the necessary database operations.
* (4) If everything works fine, then we are committing the transaction.
* (5) If any exception occurred, then we roll back the transaction.
* (6) As we are using try-with-resources syntax, the connection will be automatically closed.

In this implementation there is a lot of boilerplate code to handle database transaction.

Spring simplifies the database transaction handling mechanism by using Aspect Oriented Programming (AOP) behind the scenes. 

We can implement database transaction handling in a declarative approach using **@Transactional** annotation 
or programmatically using **TransactionTemplate**.

## Declarative Transaction Management using @Transactional annotation
Typically, in a 3-tier layered architecture, we will have a Controller in the web layer, 
Service in the business logic layer and Repository in the data access layer.

Generally, a "Unit Of Work" is modeled as a Service method and is considered to be a "transactional boundary".

We can add Spring's **@Transactional** annotation on a Service layer method to define the transaction scope. 
All the database operations in that method will be running in a single transaction. 

If the method is executed successfully, then the transaction will be committed. 
If any **RuntimeException** occurs during the method execution, then the transaction will be rolled back.

```java
@Service
class UserService {
    private final JdbcClient jdbcClient;
    
    @Transactional
    void register(User user) {
        String insertSql = "...";
        jdbcClient.sql(insertSql).update();
        String updateSql = "...";
        jdbcClient.sql(updateSql).update();
    }
}
```

- You can add **@Transactional** annotation on a method level to make that particular method transactional, 
  or at class level to make all the public methods in that class as transactional.
- When you add **@Transactional** annotation on a class or on any of its methods, 
- Spring creates a Proxy for that class and applies the transaction handling logic using Spring AOP behind the scenes.

When you add **@Transactional** annotation, by default:

- The propagation level is set to REQUIRED which participates in the current transaction if it exists or starts a new transaction.
- Rolls back the transaction if any RuntimeException is thrown by the method.
- Does NOT roll back the transaction if the method throws any Checked Exception.

You can customize this behavior by specifying the **@Transactional** attributes as follows:

```java
@Service
class UserService {
    private final JdbcClient jdbcClient;
    
    @Transactional(
            propagation = Propagation.REQUIRES_NEW,
            rollbackFor = DuplicateUserException.class,
            noRollbackFor = IllegalArgumentException.class)
    void register(User user) throws DuplicateUserException {
        String sql = "...";
        jdbcClient.sql(sql).update();
    }
}

class DuplicateUserException extends Exception {}
```

In the above code snippet, we specified propagation level to be **REQUIRES_NEW** instead of the default **REQUIRED**. 
The **REQUIRES_NEW** propagation level suspends the current transaction if one exists, starts a new transaction, 
executes the database operations, commits the transaction and then resumes the previously suspended transaction.

Also, we specified to rollback the transaction if **DuplicateUserException** occurs even though it is a Checked Exception. 

And, we specified not to rollback the transaction if **IllegalArgumentException** occurs even though it is a RuntimeException.

## Programmatic Transaction Management using TransactionTemplate
Spring also provides a programmatic approach to handling database transactions using **TransactionTemplate** as follows:

```java
@Service
class UserService {
    private final JdbcClient jdbcClient;
    private final TransactionTemplate transactionTemplate;

    void register(User user) {
        transactionTemplate.execute(status -> {
            String sql = "...";
            jdbcClient.sql(sql).update();
            
            // If anything goes wrong, rollback
            //status.setRollbackOnly();

            return result;
        });
    }
}
```

## Assignment
- Add transactional handling support to **PostRepository** using **@Transactional** annotation.
- In the **PostRepository** class **createPost()** method first insert the post record and check if post content has any swear words and throw a **RuntimeException** if it does.
  Verify that the post is not inserted in the database if the RuntimeException is thrown.

---
[Home](../README.md) | [← Previous: Spring JdbcClient](50-jdbc-client.md) | [Next: Flyway Database Migrations →](70-flyway-database-migrations.md)
