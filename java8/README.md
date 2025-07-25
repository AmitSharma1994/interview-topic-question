# Java 8 Features Interview Questions

## Interface Enhancements

- Why were default methods and static methods added to interfaces?
- How do default methods help with backward compatibility?
- How are conflicts resolved when a class implements multiple interfaces with the same default method?

## Lambda Expressions

- What is a lambda expression?
- How do lambda expressions relate to functional interfaces?
- What are the benefits of using lambda expressions?

## Functional Interfaces

- What is a functional interface?
- What are the benefits of functional interfaces?
- List some common functional interfaces in Java 8

## Stream API

- What is the difference between map and flatMap?
- What is the filter operation and how is it used?
- How do streams differ from collections?
- What are intermediate and terminal operations in streams?

## Optional Class

- What is the Optional class and why was it introduced?
- How does Optional help avoid NullPointerExceptions?
- What are the key methods in the Optional class?

## Date and Time API

- What improvements were made to the Date API in Java 8?
- How is the new Date API better than the old one?
- What are the key classes in the new Date API?

## Other Enhancements

- What is Metaspace in Java 8?
- How is it different from PermGen space?
- What improvements were made to HashMap in Java 8?

## Code Examples

```java
// Example: Using streams to filter and map a list
List<String> names = Arrays.asList("John", "Jane", "Adam", "Eve", "Mike");

List<String> filteredNames = names.stream()
    .filter(name -> name.length() > 3)
    .map(String::toUpperCase)
    .collect(Collectors.toList());

System.out.println(filteredNames); // [JOHN, JANE, ADAM, MIKE]
```

## Resources

- [Java 8 Features Resources](https://drive.google.com/drive/u/1/folders/17qqhXFi7oeXL02HG3I9eSDM0C-cOUBCI)