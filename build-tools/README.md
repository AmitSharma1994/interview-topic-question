# Build Tools Interview Questions

## Maven

- What is Maven?
- What is the lifecycle of Maven?
- What is the command for executing specific test cases?
- What does the Maven test command do?
- What does Maven clean install do?

## Gradle

- What is Gradle?
- How is Gradle different from Maven?
- What are the key features of Gradle?
- How to define dependencies in Gradle?

## Ant

- What is Apache Ant?
- How is Ant different from Maven and Gradle?
- When would you use Ant over other build tools?

## Build Lifecycle

- What are the different phases in a build lifecycle?
- How to customize the build lifecycle?
- How to skip specific phases in the build lifecycle?

## Dependency Management

- How to manage dependencies in Maven?
- What is dependency scope in Maven?
- How to resolve dependency conflicts?

## Plugins

- What are Maven plugins?
- How to configure plugins in Maven?
- What are some common Maven plugins?

## Multi-Module Projects

- How to set up a multi-module project in Maven?
- How to manage dependencies between modules?
- What are the benefits of multi-module projects?

## Code Examples

```xml
<!-- Example: Maven pom.xml -->
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>my-app</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>jar</packaging>

    <name>My Application</name>
    <description>A sample Maven project</description>

    <properties>
        <java.version>11</java.version>
        <maven.compiler.source>${java.version}</maven.compiler.source>
        <maven.compiler.target>${java.version}</maven.compiler.target>
        <spring.version>5.3.10</spring.version>
        <junit.version>5.8.1</junit.version>
    </properties>

    <dependencies>
        <!-- Spring Framework -->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>${spring.version}</version>
        </dependency>
        
        <!-- Testing -->
        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter-api</artifactId>
            <version>${junit.version}</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter-engine</artifactId>
            <version>${junit.version}</version>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <!-- Maven Compiler Plugin -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.8.1</version>
                <configuration>
                    <source>${java.version}</source>
                    <target>${java.version}</target>
                </configuration>
            </plugin>
            
            <!-- Maven Surefire Plugin for running tests -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-surefire-plugin</artifactId>
                <version>3.0.0-M5</version>
            </plugin>
            
            <!-- Maven JAR Plugin -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-jar-plugin</artifactId>
                <version>3.2.0</version>
                <configuration>
                    <archive>
                        <manifest>
                            <addClasspath>true</addClasspath>
                            <mainClass>com.example.App</mainClass>
                        </manifest>
                    </archive>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

## Resources

- [Build Tools Resources](https://docs.google.com/document/d/19y0nEIH2IsZwINlJoxWPuFOoVmN2FfshoLqZSYrnaPw/edit)