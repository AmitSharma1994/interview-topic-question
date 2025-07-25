# Application Servers Interview Questions

## Server Types

- What are the different types of application servers?
- What is the difference between web servers and application servers?
- What is the difference between blocking and non-blocking servers?

## Tomcat

- Is Tomcat a blocking or non-blocking server?
- What are the key components of Tomcat?
- How to configure Tomcat for production use?

## JBoss/WildFly

- How to deploy an application to JBoss server?
- What are the key features of JBoss/WildFly?
- How to configure JBoss for clustering?

## WebLogic

- How to deploy an application to WebLogic server?
- What are the key features of WebLogic?
- How to configure WebLogic for high availability?

## Server Configuration

- How to tune application servers for performance?
- How to configure JNDI resources?
- How to set up data sources in different application servers?

## Deployment

- What are the different deployment methods for Java applications?
- How to perform hot deployment?
- What is zero-downtime deployment?

## Code Examples

```xml
<!-- Example: Tomcat context.xml configuration -->
<Context>
    <!-- Resource definition for a database -->
    <Resource name="jdbc/MyDB"
              auth="Container"
              type="javax.sql.DataSource"
              maxTotal="100"
              maxIdle="30"
              maxWaitMillis="10000"
              username="dbuser"
              password="dbpassword"
              driverClassName="com.mysql.jdbc.Driver"
              url="jdbc:mysql://localhost:3306/mydb"/>
</Context>
```

## Resources

- [Application Server Resources](https://tomcat.apache.org/tomcat-9.0-doc/index.html)
- [JBoss Documentation](https://docs.jboss.org/)