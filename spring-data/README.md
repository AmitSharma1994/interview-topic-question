# Spring Data Interview Questions

## Spring Data Overview

- What is Spring Data?
- What are the different Spring Data modules?

## Spring JDBC

- What is Spring Data JDBC?
- How is JdbcTemplate used in Spring?
- What are the advantages of using JdbcTemplate?

## Spring Data JPA

- What is Spring Data JPA?
- How does Spring Data JPA provide a wrapper on Hibernate?
- What is the difference between JpaRepository and CrudRepository?

## Repository Methods

- What methods are available in JpaRepository? (findById, findAll, save, update, etc.)
- How can we define our own methods in repositories? (findByName, findByNameAndEmail, etc.)
- How do method name queries work in Spring Data JPA?

## Data Operations

- How to insert bulk data in JDBC, Hibernate, and Spring Data JPA?
- How to select a large number of records (e.g., 100,000) and apply pagination?

## Database Connections

- How to connect to databases using Spring Boot?
- How to configure database connection properties?
- How to connect to multiple databases in a Spring Boot application?

## Code Examples

```java
// Example: Spring Data JPA Repository with custom queries
@Repository
public interface UserRepository extends JpaRepository<User, Long> {
    
    // Query derived from method name
    List<User> findByLastName(String lastName);
    
    // Custom query using @Query annotation
    @Query("SELECT u FROM User u WHERE u.email = :email")
    Optional<User> findByEmail(@Param("email") String email);
    
    // Native SQL query
    @Query(value = "SELECT * FROM users WHERE active = true", nativeQuery = true)
    List<User> findAllActiveUsers();
    
    // Paging and sorting
    Page<User> findByCreatedDateAfter(LocalDate date, Pageable pageable);
}
```

## Resources

- [Spring Data Resources](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/)