# Spring Security Interview Questions

## Core Concepts

- How does Spring Security work internally?
- What are the different components of Spring Security?
  - Authentication Filter
  - Principal
  - Authentication Manager
  - Authentication Provider
  - UserDetailsService

## Authentication

- Which class is used to validate user ID and password?
- How are user ID and password encrypted?
- What encryption algorithms are commonly used? (BCryptPasswordEncoder, AES, DES)
- How is database user ID and password encrypted?
- How to authenticate every request of the user?

## OAuth2

- What is the OAuth2 Framework?
- What are the key components of OAuth2?
  - Authorization Server
  - Resource Server
  - Client
- What is JWT (JSON Web Token)?
- What is JWT (JSON Web Token) and OAuth2
- How to validate bearer tokens?
- How to validate the same bearer token if sent again?

## Security Best Practices

- How would you secure a Spring Boot application against common vulnerabilities?
  - SQL Injection
  - Cross-Site Scripting (XSS)
  - Cross-Site Request Forgery (CSRF)
- How would you implement authentication and authorization in a Spring Boot application?

## Code Examples

```java
// Example: Basic Spring Security Configuration
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    
    private final UserDetailsService userDetailsService;
    
    public SecurityConfig(UserDetailsService userDetailsService) {
        this.userDetailsService = userDetailsService;
    }
    
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
            .authorizeRequests()
                .antMatchers("/", "/home", "/public/**").permitAll()
                .antMatchers("/admin/**").hasRole("ADMIN")
                .anyRequest().authenticated()
                .and()
            .formLogin()
                .loginPage("/login")
                .permitAll()
                .and()
            .logout()
                .permitAll()
                .and()
            .csrf();
    }
    
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.userDetailsService(userDetailsService)
            .passwordEncoder(passwordEncoder());
    }
    
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}
```

## Resources

- [Spring Security Internal Working](https://www.linkedin.com/pulse/how-does-spring-security-works-internally-ayush-jain/)
- [OAuth2 with Spring Security](https://www.linkedin.com/pulse/oauth-spring-security-ayush-jain/)