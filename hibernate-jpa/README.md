# Hibernate and JPA Interview Questions

## Core Concepts

- What is ORM (Object-Relational Mapping)?
- What is Hibernate and how does it relate to JPA?
- What are the advantages of using Hibernate?

## Entity Mapping

- How to use composite keys in Hibernate?
- What is the difference between @Id and @EmbeddedId?
- What is the generated strategy for ID and what options are available?

## Operations

- What is the difference between save, saveOrUpdate, and persist?
- What is the difference between load and get methods?
- How to implement pagination in Hibernate?

## Caching

- What is first-level cache (session) in Hibernate?
- What is second-level cache (sessionFactory) in Hibernate?
- How to implement second-level cache?
- What caching providers are supported by Hibernate?

## Relationships

- What are the different types of relationships in Hibernate?
  - @OneToOne
  - @OneToMany
  - @ManyToOne
  - @ManyToMany
- What happens if we delete a parent entity with child entities?
- What are the different types of cascade operations?

## Inheritance Mapping

- How to save parent and child classes in inheritance mapping?
- What are the different inheritance mapping strategies?
  - Single Table
  - Joined Table
  - Table Per Class

## Annotations

- What are the common annotations used in Hibernate?
- How to map entity relationships using annotations?

## Code Examples

```java
// Example: Entity with relationships
@Entity
@Table(name = "departments")
public class Department {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String name;
    
    @OneToMany(mappedBy = "department", cascade = CascadeType.ALL, orphanRemoval = true)
    private List<Employee> employees = new ArrayList<>();
    
    // Constructors, getters, setters
    
    public void addEmployee(Employee employee) {
        employees.add(employee);
        employee.setDepartment(this);
    }
    
    public void removeEmployee(Employee employee) {
        employees.remove(employee);
        employee.setDepartment(null);
    }
}

@Entity
@Table(name = "employees")
public class Employee {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String name;
    private String email;
    
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "department_id")
    private Department department;
    
    // Constructors, getters, setters
}
```

## Resources

- [Hibernate and JPA Resources](https://drive.google.com/drive/u/1/folders/1h40wud_KNsyYKgttvicHwQIjUmAyIRn6)