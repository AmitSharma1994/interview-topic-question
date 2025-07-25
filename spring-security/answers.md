# Spring Security Interview Questions and Answers

## Core Concepts

### How does Spring Security work internally?

**Answer:**
Spring Security works through a chain of servlet filters that process the HTTP request before it reaches the application:

1. **Filter Chain:**
   - Spring Security is implemented as a series of filters
   - Each filter has a specific responsibility in the security process
   - Filters are executed in a specific order

2. **Request Processing Flow:**
   - HTTP request arrives at the server
   - Request passes through the filter chain
   - Authentication and authorization checks are performed
   - If successful, request proceeds to the application
   - If unsuccessful, appropriate error responses are returned

3. **Core Process:**
   - **Authentication:** Verifies the identity of the user
   - **Authorization:** Determines if the authenticated user has permission to access the requested resource
   - **Exception Handling:** Manages security-related exceptions

4. **SecurityContextHolder:**
   - Stores the security context for the current thread
   - Contains the Authentication object representing the current user
   - Uses ThreadLocal by default to store context

5. **SecurityContext:**
   - Contains the Authentication object
   - Available throughout the request processing

**Example flow:**
```
HTTP Request → 
  FilterChainProxy → 
    SecurityContextPersistenceFilter (restores SecurityContext) →
    UsernamePasswordAuthenticationFilter (processes login) →
    ExceptionTranslationFilter (catches security exceptions) →
    FilterSecurityInterceptor (makes access control decisions) →
  Protected Resource
```

### What are the different components of Spring Security?

**Answer:**

#### Authentication Filter
- Intercepts authentication requests (e.g., login form submissions)
- Extracts credentials from the request
- Delegates to the Authentication Manager
- Creates Authentication object upon successful authentication
- Stores Authentication in SecurityContext
- Common implementations: UsernamePasswordAuthenticationFilter, BasicAuthenticationFilter, OAuth2LoginAuthenticationFilter

**Example:**
```java
public class CustomAuthenticationFilter extends AbstractAuthenticationProcessingFilter {
    
    public CustomAuthenticationFilter() {
        super(new AntPathRequestMatcher("/login", "POST"));
    }
    
    @Override
    public Authentication attemptAuthentication(HttpServletRequest request, 
                                               HttpServletResponse response) 
                                               throws AuthenticationException {
        String username = request.getParameter("username");
        String password = request.getParameter("password");
        
        UsernamePasswordAuthenticationToken authRequest = 
            new UsernamePasswordAuthenticationToken(username, password);
            
        return this.getAuthenticationManager().authenticate(authRequest);
    }
}
```

#### Principal
- Represents the authenticated user
- Core part of the Authentication object
- Can be a username (String) or a UserDetails object
- Contains user identity and additional attributes
- Accessible via SecurityContextHolder

**Example:**
```java
Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
Object principal = authentication.getPrincipal();

if (principal instanceof UserDetails) {
    String username = ((UserDetails)principal).getUsername();
} else {
    String username = principal.toString();
}
```

#### Authentication Manager
- Central interface for authentication
- Processes Authentication requests
- Delegates to one or more AuthenticationProvider instances
- Returns fully populated Authentication object if successful
- Throws AuthenticationException if authentication fails
- Default implementation: ProviderManager

**Example:**
```java
@Configuration
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.authenticationProvider(customAuthenticationProvider())
            .authenticationProvider(anotherAuthenticationProvider());
    }
    
    @Bean
    public AuthenticationProvider customAuthenticationProvider() {
        // Custom authentication provider implementation
        return new CustomAuthenticationProvider();
    }
}
```

#### Authentication Provider
- Performs specific types of authentication
- Verifies credentials against a specific source
- Can support different authentication mechanisms
- Common implementations: DaoAuthenticationProvider, LdapAuthenticationProvider
- Custom providers can be implemented for specific needs

**Example:**
```java
@Component
public class CustomAuthenticationProvider implements AuthenticationProvider {
    
    private final UserDetailsService userDetailsService;
    private final PasswordEncoder passwordEncoder;
    
    public CustomAuthenticationProvider(UserDetailsService userDetailsService, 
                                       PasswordEncoder passwordEncoder) {
        this.userDetailsService = userDetailsService;
        this.passwordEncoder = passwordEncoder;
    }
    
    @Override
    public Authentication authenticate(Authentication authentication) 
            throws AuthenticationException {
        String username = authentication.getName();
        String password = authentication.getCredentials().toString();
        
        UserDetails userDetails = userDetailsService.loadUserByUsername(username);
        
        if (passwordEncoder.matches(password, userDetails.getPassword())) {
            return new UsernamePasswordAuthenticationToken(
                userDetails, password, userDetails.getAuthorities());
        } else {
            throw new BadCredentialsException("Invalid password");
        }
    }
    
    @Override
    public boolean supports(Class<?> authentication) {
        return UsernamePasswordAuthenticationToken.class.isAssignableFrom(authentication);
    }
}
```

#### UserDetailsService
- Loads user-specific data
- Core interface for retrieving user details
- Bridges the gap between Spring Security and user data store
- Returns UserDetails object containing username, password, authorities
- Can be implemented to retrieve users from any data source

**Example:**
```java
@Service
public class CustomUserDetailsService implements UserDetailsService {
    
    private final UserRepository userRepository;
    
    public CustomUserDetailsService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
    
    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        User user = userRepository.findByUsername(username)
            .orElseThrow(() -> new UsernameNotFoundException("User not found: " + username));
            
        return org.springframework.security.core.userdetails.User
            .withUsername(user.getUsername())
            .password(user.getPassword())
            .authorities(getAuthorities(user))
            .accountExpired(!user.isActive())
            .accountLocked(!user.isActive())
            .credentialsExpired(!user.isActive())
            .disabled(!user.isActive())
            .build();
    }
    
    private Collection<? extends GrantedAuthority> getAuthorities(User user) {
        return user.getRoles().stream()
            .map(role -> new SimpleGrantedAuthority("ROLE_" + role.getName()))
            .collect(Collectors.toList());
    }
}
```

## Authentication

### Which class is used to validate user ID and password?

**Answer:**
Several classes are involved in validating user credentials in Spring Security:

1. **AuthenticationManager:**
   - Central interface for authentication
   - Delegates to AuthenticationProvider(s)

2. **AuthenticationProvider:**
   - Performs the actual validation
   - Most common implementation is DaoAuthenticationProvider

3. **DaoAuthenticationProvider:**
   - Validates username and password
   - Uses UserDetailsService to load user data
   - Uses PasswordEncoder to verify password

4. **UserDetailsService:**
   - Loads user-specific data
   - Returns UserDetails object with credentials and authorities

5. **PasswordEncoder:**
   - Handles password encoding and verification
   - Compares submitted password with stored password

**Example:**
```java
@Configuration
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    
    @Autowired
    private UserDetailsService userDetailsService;
    
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

### How are user ID and password encrypted?

**Answer:**
In Spring Security, passwords are not encrypted but hashed using one-way cryptographic hash functions:

1. **Password Encoding Process:**
   - Passwords are never stored in plain text
   - Passwords are hashed using a one-way function
   - Many implementations add a random salt to prevent rainbow table attacks
   - Modern encoders use adaptive hashing functions with work factors

2. **PasswordEncoder Interface:**
   - Core interface for password encoding
   - Methods: encode(password) and matches(rawPassword, encodedPassword)
   - Various implementations available

3. **Common PasswordEncoder Implementations:**
   - BCryptPasswordEncoder (recommended)
   - Pbkdf2PasswordEncoder
   - SCryptPasswordEncoder
   - Argon2PasswordEncoder
   - DelegatingPasswordEncoder (default since Spring Security 5)

**Example:**
```java
@Bean
public PasswordEncoder passwordEncoder() {
    // Create BCrypt password encoder with strength 12
    return new BCryptPasswordEncoder(12);
}

// Usage
public void registerUser(User user) {
    // Hash the password before storing
    String hashedPassword = passwordEncoder.encode(user.getPassword());
    user.setPassword(hashedPassword);
    userRepository.save(user);
}

// Password verification
public boolean verifyPassword(String rawPassword, String encodedPassword) {
    return passwordEncoder.matches(rawPassword, encodedPassword);
}
```

### What encryption algorithms are commonly used? (BCryptPasswordEncoder, AES, DES)

**Answer:**
Spring Security uses various algorithms for password hashing:

1. **BCryptPasswordEncoder:**
   - Most commonly used and recommended
   - Adaptive hashing function based on Blowfish cipher
   - Automatically generates and stores salt
   - Configurable work factor (strength parameter)
   - Slower by design to resist brute force attacks

2. **Pbkdf2PasswordEncoder:**
   - Based on PBKDF2 (Password-Based Key Derivation Function 2)
   - NIST recommended
   - Configurable iterations, hash width, and secret key
   - Good for compliance requirements

3. **SCryptPasswordEncoder:**
   - Memory-hard function designed to be resistant to hardware attacks
   - More resource-intensive than BCrypt
   - Configurable CPU and memory cost parameters

4. **Argon2PasswordEncoder:**
   - Winner of the Password Hashing Competition (2015)
   - Highly secure but more resource-intensive
   - Configurable memory, iterations, and parallelism parameters
   - Available since Spring Security 5.3

5. **StandardPasswordEncoder (deprecated):**
   - SHA-256 with random salt
   - Not recommended for new applications

6. **NoOpPasswordEncoder (for testing only):**
   - Stores passwords in plain text
   - Never use in production

7. **DelegatingPasswordEncoder:**
   - Default since Spring Security 5
   - Delegates to appropriate encoder based on password format prefix
   - Supports upgrading encoding strategies while maintaining backward compatibility

**Note:** AES and DES are encryption algorithms (reversible), not password hashing algorithms. They are not used for password storage in Spring Security.

**Example:**
```java
// BCrypt (recommended for most cases)
@Bean
public PasswordEncoder passwordEncoder() {
    return new BCryptPasswordEncoder();
}

// PBKDF2
@Bean
public PasswordEncoder passwordEncoder() {
    return new Pbkdf2PasswordEncoder("secret", 16, 310000, 256);
}

// SCrypt
@Bean
public PasswordEncoder passwordEncoder() {
    return new SCryptPasswordEncoder(16384, 8, 1, 32, 64);
}

// Argon2
@Bean
public PasswordEncoder passwordEncoder() {
    return new Argon2PasswordEncoder(16, 32, 1, 65536, 10);
}

// DelegatingPasswordEncoder (default in Spring Security 5+)
@Bean
public PasswordEncoder passwordEncoder() {
    return PasswordEncoderFactories.createDelegatingPasswordEncoder();
}
```

### How is database user ID and password encrypted?

**Answer:**
In a Spring Security application with database authentication:

1. **Password Storage:**
   - Passwords are hashed (not encrypted) before storage
   - Only the hash is stored in the database
   - Original password cannot be recovered from the hash
   - Format typically includes algorithm identifier: `{bcrypt}$2a$10$...`

2. **Authentication Process:**
   - User submits credentials
   - System loads stored user details including hashed password
   - Submitted password is hashed using same algorithm
   - Hashes are compared to verify password

3. **Implementation Steps:**
   - Configure a PasswordEncoder bean
   - Use the encoder when saving user passwords
   - Configure UserDetailsService to load user data from database
   - Spring Security handles password verification automatically

**Example:**
```java
// User entity
@Entity
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String username;
    private String password; // Stores hashed password
    
    // Other fields, getters, setters
}

// User repository
@Repository
public interface UserRepository extends JpaRepository<User, Long> {
    Optional<User> findByUsername(String username);
}

// User service for registration
@Service
public class UserService {
    private final UserRepository userRepository;
    private final PasswordEncoder passwordEncoder;
    
    public UserService(UserRepository userRepository, PasswordEncoder passwordEncoder) {
        this.userRepository = userRepository;
        this.passwordEncoder = passwordEncoder;
    }
    
    public User registerUser(String username, String rawPassword) {
        User user = new User();
        user.setUsername(username);
        // Hash password before storing
        user.setPassword(passwordEncoder.encode(rawPassword));
        return userRepository.save(user);
    }
}

// UserDetailsService implementation
@Service
public class DatabaseUserDetailsService implements UserDetailsService {
    private final UserRepository userRepository;
    
    public DatabaseUserDetailsService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
    
    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        User user = userRepository.findByUsername(username)
            .orElseThrow(() -> new UsernameNotFoundException("User not found"));
            
        return org.springframework.security.core.userdetails.User
            .withUsername(user.getUsername())
            .password(user.getPassword()) // Already hashed
            .authorities("ROLE_USER")
            .build();
    }
}

// Security configuration
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    
    private final UserDetailsService userDetailsService;
    
    public SecurityConfig(UserDetailsService userDetailsService) {
        this.userDetailsService = userDetailsService;
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

### How to authenticate every request of the user?

**Answer:**
Spring Security provides several mechanisms to authenticate every request:

1. **Session-Based Authentication:**
   - Default approach in Spring Security
   - User authenticates once (e.g., login form)
   - Session is created with authentication details
   - SecurityContextPersistenceFilter restores authentication on subsequent requests
   - JSESSIONID cookie tracks the session

2. **Token-Based Authentication:**
   - JWT or other token formats
   - Token included in each request (typically in Authorization header)
   - Custom filter extracts and validates token
   - Creates Authentication object for the request

3. **Basic Authentication:**
   - Credentials sent in Authorization header with each request
   - Format: `Authorization: Basic base64(username:password)`
   - Enabled with http.httpBasic()

4. **Remember-Me Authentication:**
   - Persists authentication between browser sessions
   - Uses cookies to remember the user
   - Configured with http.rememberMe()

5. **OAuth2/OpenID Connect:**
   - Uses access tokens for authentication
   - Configured with OAuth2LoginConfigurer or ResourceServerConfigurer

**Example configurations:**

**Session-based authentication:**
```java
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
            .authorizeRequests()
                .antMatchers("/public/**").permitAll()
                .anyRequest().authenticated()
                .and()
            .formLogin()
                .loginPage("/login")
                .permitAll()
                .and()
            .logout()
                .permitAll()
                .and()
            .sessionManagement()
                .sessionCreationPolicy(SessionCreationPolicy.IF_REQUIRED);
    }
}
```

**JWT token authentication:**
```java
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    
    private final JwtTokenFilter jwtTokenFilter;
    
    public SecurityConfig(JwtTokenFilter jwtTokenFilter) {
        this.jwtTokenFilter = jwtTokenFilter;
    }
    
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
            .csrf().disable()
            .sessionManagement()
                .sessionCreationPolicy(SessionCreationPolicy.STATELESS)
                .and()
            .authorizeRequests()
                .antMatchers("/auth/**").permitAll()
                .anyRequest().authenticated();
                
        // Add JWT token filter
        http.addFilterBefore(jwtTokenFilter, UsernamePasswordAuthenticationFilter.class);
    }
}

// JWT Token Filter
@Component
public class JwtTokenFilter extends OncePerRequestFilter {
    
    private final JwtTokenProvider tokenProvider;
    
    public JwtTokenFilter(JwtTokenProvider tokenProvider) {
        this.tokenProvider = tokenProvider;
    }
    
    @Override
    protected void doFilterInternal(HttpServletRequest request, 
                                   HttpServletResponse response, 
                                   FilterChain filterChain) 
                                   throws ServletException, IOException {
        String token = getTokenFromRequest(request);
        
        if (token != null && tokenProvider.validateToken(token)) {
            Authentication auth = tokenProvider.getAuthentication(token);
            SecurityContextHolder.getContext().setAuthentication(auth);
        }
        
        filterChain.doFilter(request, response);
    }
    
    private String getTokenFromRequest(HttpServletRequest request) {
        String bearerToken = request.getHeader("Authorization");
        if (bearerToken != null && bearerToken.startsWith("Bearer ")) {
            return bearerToken.substring(7);
        }
        return null;
    }
}
```

## OAuth2

### What is the OAuth2 Framework?

**Answer:**
OAuth 2.0 is an authorization framework that enables third-party applications to obtain limited access to a user's account on a server:

1. **Core Purpose:**
   - Delegated authorization framework
   - Allows applications to access resources on behalf of users
   - Separates authentication from authorization
   - Eliminates need to share credentials with third parties

2. **Key Characteristics:**
   - Token-based authorization
   - Different grant types for different scenarios
   - Scoped permissions (limited access)
   - Separation of roles (client, resource server, authorization server)

3. **Benefits:**
   - Improved security (no credential sharing)
   - Standardized protocol
   - Fine-grained access control
   - Revocable access
   - Wide industry adoption

4. **Common Use Cases:**
   - "Login with Google/Facebook/GitHub"
   - API authorization
   - Microservices security
   - Mobile app authentication

**Example flow (Authorization Code Grant):**
1. User wants to use an application that needs access to their Google Drive
2. Application redirects user to Google's authorization server
3. User authenticates with Google and approves the requested permissions
4. Google redirects back to the application with an authorization code
5. Application exchanges the code for an access token
6. Application uses the access token to access the user's Google Drive

### What are the key components of OAuth2?

**Answer:**

#### Authorization Server
- Issues access tokens to clients
- Authenticates resource owners (users)
- Obtains authorization from resource owners
- Maintains token information
- Provides endpoints:
  - Authorization endpoint: User authorizes client access
  - Token endpoint: Client obtains tokens
  - Introspection endpoint: Validates tokens
  - Revocation endpoint: Revokes tokens

**Example in Spring Security:**
```java
@Configuration
@EnableAuthorizationServer
public class AuthServerConfig extends AuthorizationServerConfigurerAdapter {
    
    private final AuthenticationManager authenticationManager;
    private final PasswordEncoder passwordEncoder;
    private final UserDetailsService userDetailsService;
    
    public AuthServerConfig(AuthenticationManager authenticationManager,
                           PasswordEncoder passwordEncoder,
                           UserDetailsService userDetailsService) {
        this.authenticationManager = authenticationManager;
        this.passwordEncoder = passwordEncoder;
        this.userDetailsService = userDetailsService;
    }
    
    @Override
    public void configure(ClientDetailsServiceConfigurer clients) throws Exception {
        clients.inMemory()
            .withClient("client-id")
            .secret(passwordEncoder.encode("client-secret"))
            .authorizedGrantTypes("authorization_code", "refresh_token", "password")
            .scopes("read", "write")
            .redirectUris("http://localhost:8080/callback")
            .accessTokenValiditySeconds(3600);
    }
    
    @Override
    public void configure(AuthorizationServerEndpointsConfigurer endpoints) {
        endpoints
            .authenticationManager(authenticationManager)
            .userDetailsService(userDetailsService);
    }
}
```

#### Resource Server
- Hosts protected resources
- Validates access tokens
- Enforces authorization rules
- Serves resources when valid token is presented
- Can communicate with authorization server to validate tokens

**Example in Spring Security:**
```java
@Configuration
@EnableResourceServer
public class ResourceServerConfig extends ResourceServerConfigurerAdapter {
    
    @Override
    public void configure(HttpSecurity http) throws Exception {
        http
            .authorizeRequests()
                .antMatchers("/api/public/**").permitAll()
                .antMatchers("/api/user/**").hasRole("USER")
                .antMatchers("/api/admin/**").hasRole("ADMIN")
                .anyRequest().authenticated();
    }
    
    @Override
    public void configure(ResourceServerSecurityConfigurer resources) {
        resources.resourceId("api");
    }
}
```

#### Client
- Application requesting access to resources
- Obtains authorization from resource owner
- Makes requests to resource server with access token
- Stores and manages tokens
- Types:
  - Confidential clients (can securely store secrets)
  - Public clients (cannot securely store secrets)

**Example client implementation:**
```java
@Service
public class OAuth2Client {
    
    private final RestTemplate restTemplate;
    private final String clientId;
    private final String clientSecret;
    private final String authorizationServerUrl;
    
    public OAuth2Client(RestTemplate restTemplate,
                       @Value("${oauth2.client.id}") String clientId,
                       @Value("${oauth2.client.secret}") String clientSecret,
                       @Value("${oauth2.auth-server.url}") String authorizationServerUrl) {
        this.restTemplate = restTemplate;
        this.clientId = clientId;
        this.clientSecret = clientSecret;
        this.authorizationServerUrl = authorizationServerUrl;
    }
    
    public String getAuthorizationUrl(String redirectUri, String scope) {
        return UriComponentsBuilder.fromHttpUrl(authorizationServerUrl + "/oauth/authorize")
            .queryParam("client_id", clientId)
            .queryParam("response_type", "code")
            .queryParam("redirect_uri", redirectUri)
            .queryParam("scope", scope)
            .toUriString();
    }
    
    public TokenResponse exchangeCodeForToken(String code, String redirectUri) {
        HttpHeaders headers = new HttpHeaders();
        headers.setBasicAuth(clientId, clientSecret);
        headers.setContentType(MediaType.APPLICATION_FORM_URLENCODED);
        
        MultiValueMap<String, String> map = new LinkedMultiValueMap<>();
        map.add("grant_type", "authorization_code");
        map.add("code", code);
        map.add("redirect_uri", redirectUri);
        
        HttpEntity<MultiValueMap<String, String>> request = new HttpEntity<>(map, headers);
        
        return restTemplate.postForObject(
            authorizationServerUrl + "/oauth/token", 
            request, 
            TokenResponse.class);
    }
}
```

### What is JWT (JSON Web Token)?

**Answer:**
JWT (JSON Web Token) is a compact, self-contained method for securely transmitting information between parties as a JSON object:

1. **Structure:**
   - Three parts separated by dots: `header.payload.signature`
   - **Header:** Contains token type and signing algorithm
   - **Payload:** Contains claims (statements about an entity)
   - **Signature:** Verifies the token hasn't been altered

2. **Key Characteristics:**
   - Self-contained: Contains all necessary information
   - Stateless: No need to store session data on server
   - Compact: Can be sent through URL, POST parameter, or HTTP header
   - Signed: Ensures integrity of the data
   - Optionally encrypted: Can protect sensitive information

3. **Common Claims:**
   - **iss** (Issuer): Who issued the token
   - **sub** (Subject): Who the token refers to
   - **exp** (Expiration Time): When the token expires
   - **iat** (Issued At): When the token was issued
   - **aud** (Audience): Who the token is intended for
   - Custom claims: Any additional information

4. **Benefits:**
   - Reduced database queries (self-contained)
   - Scalability (stateless)
   - Decentralized validation
   - Cross-domain/cross-service authentication

5. **Limitations:**
   - Cannot be revoked before expiration
   - Size limitations (should be kept small)
   - Sensitive data should not be included unless encrypted

**Example JWT structure:**
```
// Header (Algorithm & Token Type)
{
  "alg": "HS256",
  "typ": "JWT"
}

// Payload (Claims)
{
  "sub": "1234567890",
  "name": "John Doe",
  "admin": true,
  "iat": 1516239022,
  "exp": 1516242622
}

// Signature
HMACSHA256(
  base64UrlEncode(header) + "." +
  base64UrlEncode(payload),
  secret
)
```

**Example JWT implementation in Spring Security:**
```java
@Component
public class JwtTokenProvider {
    
    @Value("${jwt.secret}")
    private String jwtSecret;
    
    @Value("${jwt.expiration}")
    private long jwtExpiration;
    
    public String generateToken(Authentication authentication) {
        UserDetails userDetails = (UserDetails) authentication.getPrincipal();
        Date now = new Date();
        Date expiryDate = new Date(now.getTime() + jwtExpiration);
        
        return Jwts.builder()
            .setSubject(userDetails.getUsername())
            .setIssuedAt(now)
            .setExpiration(expiryDate)
            .claim("authorities", getAuthorities(userDetails))
            .signWith(SignatureAlgorithm.HS512, jwtSecret)
            .compact();
    }
    
    public String getUsernameFromToken(String token) {
        Claims claims = Jwts.parser()
            .setSigningKey(jwtSecret)
            .parseClaimsJws(token)
            .getBody();
            
        return claims.getSubject();
    }
    
    public boolean validateToken(String token) {
        try {
            Jwts.parser().setSigningKey(jwtSecret).parseClaimsJws(token);
            return true;
        } catch (Exception e) {
            return false;
        }
    }
    
    private List<String> getAuthorities(UserDetails userDetails) {
        return userDetails.getAuthorities().stream()
            .map(GrantedAuthority::getAuthority)
            .collect(Collectors.toList());
    }
}
```

### What is JWT (JSON Web Token) and OAuth2

**Answer:**
JWT and OAuth2 are complementary technologies often used together in modern authentication and authorization systems:

1. **Relationship:**
   - OAuth2 is an authorization framework
   - JWT is a token format
   - JWT can be used as the token format within OAuth2
   - OAuth2 can use other token formats besides JWT

2. **How They Work Together:**
   - OAuth2 defines the authorization flows and roles
   - JWT provides a standardized token format
   - OAuth2 access tokens can be implemented as JWTs
   - JWT enables stateless validation of OAuth2 tokens

3. **Benefits of Using JWT with OAuth2:**
   - Self-contained tokens with claims about the user
   - Reduced database lookups for token validation
   - Cross-service authentication without shared sessions
   - Ability to include authorization data in the token itself
   - Standardized token validation

4. **Implementation Patterns:**
   - **Authorization Server:** Issues JWT access tokens
   - **Resource Server:** Validates JWT tokens locally
   - **Token Introspection:** Optional validation against auth server
   - **Token Refresh:** Using refresh tokens to obtain new JWTs

**Example OAuth2 with JWT in Spring Security:**
```java
// Authorization Server Configuration
@Configuration
@EnableAuthorizationServer
public class AuthServerConfig extends AuthorizationServerConfigurerAdapter {
    
    private final AuthenticationManager authenticationManager;
    private final JwtAccessTokenConverter jwtAccessTokenConverter;
    private final TokenStore tokenStore;
    
    public AuthServerConfig(AuthenticationManager authenticationManager,
                           JwtAccessTokenConverter jwtAccessTokenConverter,
                           TokenStore tokenStore) {
        this.authenticationManager = authenticationManager;
        this.jwtAccessTokenConverter = jwtAccessTokenConverter;
        this.tokenStore = tokenStore;
    }
    
    @Override
    public void configure(AuthorizationServerEndpointsConfigurer endpoints) {
        endpoints
            .authenticationManager(authenticationManager)
            .tokenStore(tokenStore)
            .accessTokenConverter(jwtAccessTokenConverter);
    }
    
    @Override
    public void configure(ClientDetailsServiceConfigurer clients) throws Exception {
        clients.inMemory()
            .withClient("client-id")
            .secret("{bcrypt}$2a$10$...")
            .authorizedGrantTypes("password", "refresh_token")
            .scopes("read", "write")
            .accessTokenValiditySeconds(3600);
    }
}

// JWT Configuration
@Configuration
public class JwtConfig {
    
    @Value("${jwt.key}")
    private String jwtKey;
    
    @Bean
    public TokenStore tokenStore() {
        return new JwtTokenStore(accessTokenConverter());
    }
    
    @Bean
    public JwtAccessTokenConverter accessTokenConverter() {
        JwtAccessTokenConverter converter = new JwtAccessTokenConverter();
        converter.setSigningKey(jwtKey);
        return converter;
    }
}

// Resource Server Configuration
@Configuration
@EnableResourceServer
public class ResourceServerConfig extends ResourceServerConfigurerAdapter {
    
    private final TokenStore tokenStore;
    
    public ResourceServerConfig(TokenStore tokenStore) {
        this.tokenStore = tokenStore;
    }
    
    @Override
    public void configure(ResourceServerSecurityConfigurer resources) {
        resources.tokenStore(tokenStore);
    }
    
    @Override
    public void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
            .antMatchers("/api/**").authenticated();
    }
}
```

### How to validate bearer tokens?

**Answer:**
Bearer tokens (including JWTs) can be validated in several ways in Spring Security:

1. **Token Validation Approaches:**
   - **Local validation:** Verify token signature and claims locally
   - **Remote validation:** Validate token against authorization server
   - **Hybrid approach:** Local validation with periodic remote checks

2. **JWT Token Validation Steps:**
   - Verify token signature using secret key or public key
   - Check token expiration (exp claim)
   - Validate issuer (iss claim)
   - Validate audience (aud claim)
   - Extract user details and authorities

3. **Opaque Token Validation:**
   - Call token introspection endpoint
   - Check if token is active
   - Extract scope and user information

**Example JWT token validation:**
```java
@Component
public class JwtTokenValidator {
    
    @Value("${jwt.secret}")
    private String jwtSecret;
    
    @Value("${jwt.issuer}")
    private String expectedIssuer;
    
    public boolean validateToken(String token) {
        try {
            // Parse and verify signature
            Jws<Claims> claims = Jwts.parser()
                .setSigningKey(jwtSecret)
                .parseClaimsJws(token);
                
            // Get claims
            Claims body = claims.getBody();
            
            // Check expiration
            Date expiration = body.getExpiration();
            if (expiration != null && expiration.before(new Date())) {
                return false;
            }
            
            // Check issuer
            String issuer = body.getIssuer();
            if (issuer != null && !issuer.equals(expectedIssuer)) {
                return false;
            }
            
            return true;
        } catch (Exception e) {
            return false;
        }
    }
    
    public Authentication getAuthentication(String token) {
        Claims claims = Jwts.parser()
            .setSigningKey(jwtSecret)
            .parseClaimsJws(token)
            .getBody();
            
        String username = claims.getSubject();
        List<String> authorities = claims.get("authorities", List.class);
        
        if (username != null) {
            UserDetails userDetails = User.withUsername(username)
                .password("")
                .authorities(mapToAuthorities(authorities))
                .build();
                
            return new UsernamePasswordAuthenticationToken(
                userDetails, "", userDetails.getAuthorities());
        }
        
        return null;
    }
    
    private Collection<? extends GrantedAuthority> mapToAuthorities(List<String> authorities) {
        return authorities.stream()
            .map(SimpleGrantedAuthority::new)
            .collect(Collectors.toList());
    }
}

// Using the validator in a filter
@Component
public class JwtTokenFilter extends OncePerRequestFilter {
    
    private final JwtTokenValidator tokenValidator;
    
    public JwtTokenFilter(JwtTokenValidator tokenValidator) {
        this.tokenValidator = tokenValidator;
    }
    
    @Override
    protected void doFilterInternal(HttpServletRequest request, 
                                   HttpServletResponse response, 
                                   FilterChain filterChain) 
                                   throws ServletException, IOException {
        String token = extractToken(request);
        
        if (token != null && tokenValidator.validateToken(token)) {
            Authentication auth = tokenValidator.getAuthentication(token);
            SecurityContextHolder.getContext().setAuthentication(auth);
        }
        
        filterChain.doFilter(request, response);
    }
    
    private String extractToken(HttpServletRequest request) {
        String bearerToken = request.getHeader("Authorization");
        if (bearerToken != null && bearerToken.startsWith("Bearer ")) {
            return bearerToken.substring(7);
        }
        return null;
    }
}
```

**Example OAuth2 opaque token validation:**
```java
@Configuration
@EnableResourceServer
public class ResourceServerConfig extends ResourceServerConfigurerAdapter {
    
    @Value("${oauth2.check-token-url}")
    private String checkTokenUrl;
    
    @Override
    public void configure(ResourceServerSecurityConfigurer resources) {
        resources.tokenServices(tokenServices());
    }
    
    @Bean
    public ResourceServerTokenServices tokenServices() {
        RemoteTokenServices tokenServices = new RemoteTokenServices();
        tokenServices.setCheckTokenEndpointUrl(checkTokenUrl);
        tokenServices.setClientId("resource-server");
        tokenServices.setClientSecret("secret");
        return tokenServices;
    }
}
```

### How to validate the same bearer token if sent again?

**Answer:**
Validating a bearer token that is sent multiple times requires strategies to prevent token reuse attacks and ensure token freshness:

1. **Token Validation Strategies:**
   - **Stateless validation:** Validate token signature and claims on each request
   - **Token blacklisting:** Maintain a list of revoked tokens
   - **Token fingerprinting:** Include client-specific data in token
   - **One-time tokens:** Issue tokens that can only be used once
   - **Token binding:** Bind tokens to specific contexts (e.g., TLS session)

2. **JWT Specific Approaches:**
   - Include token ID (jti claim) for uniqueness
   - Maintain a cache of used token IDs
   - Use short expiration times
   - Implement token rotation

3. **Security Considerations:**
   - Balance between security and performance
   - Consider token storage requirements
   - Evaluate impact on scalability

**Example implementation with token blacklisting:**
```java
@Service
public class TokenBlacklistService {
    
    private final Set<String> blacklistedTokens = Collections.synchronizedSet(new HashSet<>());
    private final Cache<String, Boolean> usedTokenCache;
    
    public TokenBlacklistService() {
        this.usedTokenCache = Caffeine.newBuilder()
            .expireAfterWrite(1, TimeUnit.HOURS)
            .maximumSize(10000)
            .build();
    }
    
    public void blacklistToken(String token) {
        blacklistedTokens.add(token);
    }
    
    public boolean isBlacklisted(String token) {
        return blacklistedTokens.contains(token);
    }
    
    public boolean isFirstUse(String tokenId) {
        return usedTokenCache.asMap().putIfAbsent(tokenId, Boolean.TRUE) == null;
    }
}

@Component
public class EnhancedJwtTokenValidator {
    
    private final JwtTokenValidator jwtValidator;
    private final TokenBlacklistService blacklistService;
    
    public EnhancedJwtTokenValidator(JwtTokenValidator jwtValidator,
                                    TokenBlacklistService blacklistService) {
        this.jwtValidator = jwtValidator;
        this.blacklistService = blacklistService;
    }
    
    public boolean validateToken(String token) {
        // Check if token is blacklisted
        if (blacklistService.isBlacklisted(token)) {
            return false;
        }
        
        // Basic validation (signature, expiration, etc.)
        if (!jwtValidator.validateToken(token)) {
            return false;
        }
        
        // Extract token ID
        String tokenId = extractTokenId(token);
        if (tokenId != null) {
            // Check if token has been used before (for one-time tokens)
            if (!blacklistService.isFirstUse(tokenId)) {
                return false;
            }
        }
        
        return true;
    }
    
    private String extractTokenId(String token) {
        try {
            Claims claims = Jwts.parser()
                .setSigningKey(jwtSecret)
                .parseClaimsJws(token)
                .getBody();
                
            return claims.getId(); // jti claim
        } catch (Exception e) {
            return null;
        }
    }
}
```

**Example with token binding to client fingerprint:**
```java
@Component
public class TokenBindingValidator {
    
    private final JwtTokenValidator jwtValidator;
    
    public TokenBindingValidator(JwtTokenValidator jwtValidator) {
        this.jwtValidator = jwtValidator;
    }
    
    public boolean validateToken(String token, HttpServletRequest request) {
        // Basic validation
        if (!jwtValidator.validateToken(token)) {
            return false;
        }
        
        // Extract client fingerprint from token
        String tokenFingerprint = extractClientFingerprint(token);
        if (tokenFingerprint == null) {
            return false;
        }
        
        // Generate current client fingerprint
        String currentFingerprint = generateClientFingerprint(request);
        
        // Compare fingerprints
        return tokenFingerprint.equals(currentFingerprint);
    }
    
    private String extractClientFingerprint(String token) {
        try {
            Claims claims = Jwts.parser()
                .setSigningKey(jwtSecret)
                .parseClaimsJws(token)
                .getBody();
                
            return claims.get("fingerprint", String.class);
        } catch (Exception e) {
            return null;
        }
    }
    
    private String generateClientFingerprint(HttpServletRequest request) {
        // Create fingerprint from client attributes
        String userAgent = request.getHeader("User-Agent");
        String ipAddress = request.getRemoteAddr();
        
        // Hash the fingerprint data
        return DigestUtils.sha256Hex(userAgent + ipAddress);
    }
}
```

## Security Best Practices

### How would you secure a Spring Boot application against common vulnerabilities?

**Answer:**
Securing a Spring Boot application requires addressing multiple types of vulnerabilities:

#### SQL Injection
SQL Injection occurs when untrusted data is included in SQL queries, allowing attackers to manipulate the query.

**Prevention measures:**
1. **Use parameterized queries:**
   ```java
   // UNSAFE - vulnerable to SQL injection
   String query = "SELECT * FROM users WHERE username = '" + username + "'";
   
   // SAFE - using parameterized query
   String query = "SELECT * FROM users WHERE username = ?";
   PreparedStatement stmt = connection.prepareStatement(query);
   stmt.setString(1, username);
   ```

2. **Use JPA/Hibernate:**
   ```java
   // Safe by default
   User user = userRepository.findByUsername(username);
   
   // Safe JPQL query
   @Query("SELECT u FROM User u WHERE u.username = :username")
   User findUserByUsername(@Param("username") String username);
   ```

3. **Use query builders:**
   ```java
   // Using QueryDSL
   QUser user = QUser.user;
   User result = queryFactory.selectFrom(user)
       .where(user.username.eq(username))
       .fetchOne();
   ```

4. **Input validation:**
   ```java
   @RestController
   public class UserController {
       @GetMapping("/users/{username}")
       public User getUser(@Pattern(regexp = "[a-zA-Z0-9]+") @PathVariable String username) {
           return userService.findByUsername(username);
       }
   }
   ```

#### Cross-Site Scripting (XSS)
XSS occurs when untrusted data is included in HTML output without proper encoding.

**Prevention measures:**
1. **Output encoding:**
   ```java
   // Using Spring's HtmlUtils
   String safeHtml = HtmlUtils.htmlEscape(userInput);
   
   // Using OWASP Java Encoder
   String safeHtml = Encode.forHtml(userInput);
   ```

2. **Content Security Policy (CSP):**
   ```java
   @Configuration
   public class WebSecurityConfig extends WebSecurityConfigurerAdapter {
       @Override
       protected void configure(HttpSecurity http) throws Exception {
           http.headers()
               .contentSecurityPolicy("script-src 'self'");
       }
   }
   ```

3. **Use Thymeleaf with proper context:**
   ```html
   <!-- Safe by default -->
   <p th:text="${userComment}"></p>
   
   <!-- Unsafe - only use when content is trusted -->
   <p th:utext="${userComment}"></p>
   ```

4. **XSS protection header:**
   ```java
   http.headers()
       .xssProtection()
       .block(true);
   ```

#### Cross-Site Request Forgery (CSRF)
CSRF occurs when an attacker tricks a user's browser into making an unwanted request to a site where the user is authenticated.

**Prevention measures:**
1. **Enable CSRF protection (on by default):**
   ```java
   @Configuration
   public class SecurityConfig extends WebSecurityConfigurerAdapter {
       @Override
       protected void configure(HttpSecurity http) throws Exception {
           http.csrf(); // Enabled by default
       }
   }
   ```

2. **Include CSRF token in forms:**
   ```html
   <!-- Thymeleaf -->
   <form th:action="@{/process}" method="post">
       <!-- CSRF token automatically included -->
       <input type="text" name="username" />
       <button type="submit">Submit</button>
   </form>
   ```

3. **Include CSRF token in AJAX requests:**
   ```javascript
   // Using jQuery
   $.ajax({
       url: "/api/data",
       type: "POST",
       data: JSON.stringify(data),
       beforeSend: function(xhr) {
           xhr.setRequestHeader("X-CSRF-TOKEN", $("meta[name='_csrf']").attr("content"));
       }
   });
   ```

4. **CSRF configuration for REST APIs:**
   ```java
   @Configuration
   public class SecurityConfig extends WebSecurityConfigurerAdapter {
       @Override
       protected void configure(HttpSecurity http) throws Exception {
           http
               .csrf()
                   .csrfTokenRepository(CookieCsrfTokenRepository.withHttpOnlyFalse());
       }
   }
   ```

**Additional security measures:**

1. **HTTP Security Headers:**
   ```java
   @Configuration
   public class SecurityConfig extends WebSecurityConfigurerAdapter {
       @Override
       protected void configure(HttpSecurity http) throws Exception {
           http
               .headers()
                   .contentSecurityPolicy("script-src 'self'")
                   .and()
                   .frameOptions().deny()
                   .and()
                   .xssProtection().block(true)
                   .and()
                   .httpStrictTransportSecurity()
                       .includeSubDomains(true)
                       .maxAgeInSeconds(31536000);
       }
   }
   ```

2. **Input validation with Bean Validation:**
   ```java
   public class UserRegistrationDto {
       @NotBlank
       @Size(min = 4, max = 50)
       @Pattern(regexp = "[a-zA-Z0-9]+")
       private String username;
       
       @NotBlank
       @Size(min = 8, max = 100)
       private String password;
       
       @NotBlank
       @Email
       private String email;
   }
   ```

3. **Secure password storage:**
   ```java
   @Bean
   public PasswordEncoder passwordEncoder() {
       return new BCryptPasswordEncoder(12);
   }
   ```

4. **HTTPS enforcement:**
   ```java
   @Configuration
   public class ServerConfig {
       @Bean
       public ServletWebServerFactory servletContainer() {
           TomcatServletWebServerFactory tomcat = new TomcatServletWebServerFactory();
           tomcat.addAdditionalTomcatConnectors(redirectConnector());
           return tomcat;
       }
       
       private Connector redirectConnector() {
           Connector connector = new Connector("org.apache.coyote.http11.Http11NioProtocol");
           connector.setScheme("http");
           connector.setPort(8080);
           connector.setSecure(false);
           connector.setRedirectPort(8443);
           return connector;
       }
   }
   
   // In application.properties
   server.ssl.key-store=classpath:keystore.p12
   server.ssl.key-store-password=password
   server.ssl.key-store-type=PKCS12
   server.ssl.key-alias=tomcat
   server.port=8443
   ```

5. **Rate limiting:**
   ```java
   @Component
   public class RateLimitingFilter extends OncePerRequestFilter {
       private final RateLimiter rateLimiter = RateLimiter.create(10.0); // 10 requests per second
       
       @Override
       protected void doFilterInternal(HttpServletRequest request, 
                                      HttpServletResponse response, 
                                      FilterChain filterChain) 
                                      throws ServletException, IOException {
           if (!rateLimiter.tryAcquire()) {
               response.setStatus(HttpStatus.TOO_MANY_REQUESTS.value());
               response.getWriter().write("Too many requests");
               return;
           }
           
           filterChain.doFilter(request, response);
       }
   }
   ```

### How would you implement authentication and authorization in a Spring Boot application?

**Answer:**
Implementing authentication and authorization in a Spring Boot application involves several components and configurations:

1. **Basic Authentication Setup:**
   ```java
   @Configuration
   @EnableWebSecurity
   public class SecurityConfig extends WebSecurityConfigurerAdapter {
       
       private final UserDetailsService userDetailsService;
       
       public SecurityConfig(UserDetailsService userDetailsService) {
           this.userDetailsService = userDetailsService;
       }
       
       @Override
       protected void configure(AuthenticationManagerBuilder auth) throws Exception {
           auth.userDetailsService(userDetailsService)
               .passwordEncoder(passwordEncoder());
       }
       
       @Override
       protected void configure(HttpSecurity http) throws Exception {
           http
               .authorizeRequests()
                   .antMatchers("/", "/home", "/public/**").permitAll()
                   .antMatchers("/admin/**").hasRole("ADMIN")
                   .antMatchers("/user/**").hasRole("USER")
                   .anyRequest().authenticated()
                   .and()
               .formLogin()
                   .loginPage("/login")
                   .permitAll()
                   .and()
               .logout()
                   .permitAll();
       }
       
       @Bean
       public PasswordEncoder passwordEncoder() {
           return new BCryptPasswordEncoder();
       }
   }
   ```

2. **Custom UserDetailsService:**
   ```java
   @Service
   public class CustomUserDetailsService implements UserDetailsService {
       
       private final UserRepository userRepository;
       
       public CustomUserDetailsService(UserRepository userRepository) {
           this.userRepository = userRepository;
       }
       
       @Override
       public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
           User user = userRepository.findByUsername(username)
               .orElseThrow(() -> new UsernameNotFoundException("User not found: " + username));
               
           return org.springframework.security.core.userdetails.User
               .withUsername(user.getUsername())
               .password(user.getPassword())
               .authorities(getAuthorities(user))
               .accountExpired(!user.isActive())
               .accountLocked(!user.isActive())
               .credentialsExpired(!user.isActive())
               .disabled(!user.isActive())
               .build();
       }
       
       private Collection<? extends GrantedAuthority> getAuthorities(User user) {
           return user.getRoles().stream()
               .map(role -> new SimpleGrantedAuthority("ROLE_" + role.getName()))
               .collect(Collectors.toList());
       }
   }
   ```

3. **JWT Authentication:**
   ```java
   @Configuration
   @EnableWebSecurity
   public class JwtSecurityConfig extends WebSecurityConfigurerAdapter {
       
       private final JwtTokenProvider tokenProvider;
       
       public JwtSecurityConfig(JwtTokenProvider tokenProvider) {
           this.tokenProvider = tokenProvider;
       }
       
       @Override
       protected void configure(HttpSecurity http) throws Exception {
           http
               .csrf().disable()
               .sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS)
               .and()
               .authorizeRequests()
                   .antMatchers("/auth/**").permitAll()
                   .antMatchers("/api/admin/**").hasRole("ADMIN")
                   .antMatchers("/api/user/**").hasRole("USER")
                   .anyRequest().authenticated();
                   
           http.addFilterBefore(
               new JwtTokenFilter(tokenProvider),
               UsernamePasswordAuthenticationFilter.class
           );
       }
       
       @Bean
       @Override
       public AuthenticationManager authenticationManagerBean() throws Exception {
           return super.authenticationManagerBean();
       }
   }
   ```

4. **Method-Level Security:**
   ```java
   @Configuration
   @EnableGlobalMethodSecurity(
       prePostEnabled = true,
       securedEnabled = true,
       jsr250Enabled = true)
   public class MethodSecurityConfig extends GlobalMethodSecurityConfiguration {
   }
   
   @Service
   public class UserService {
       
       @PreAuthorize("hasRole('ADMIN')")
       public List<User> getAllUsers() {
           // Only accessible to admins
           return userRepository.findAll();
       }
       
       @PostAuthorize("returnObject.username == authentication.name or hasRole('ADMIN')")
       public User getUser(Long id) {
           // Only accessible to the user themselves or admins
           return userRepository.findById(id).orElseThrow();
       }
       
       @Secured({"ROLE_USER", "ROLE_ADMIN"})
       public void updateUser(User user) {
           // Accessible to users and admins
       }
       
       @RolesAllowed("ADMIN")
       public void deleteUser(Long id) {
           // Only accessible to admins
       }
   }
   ```

5. **OAuth2 Authentication:**
   ```java
   @Configuration
   @EnableWebSecurity
   public class OAuth2SecurityConfig extends WebSecurityConfigurerAdapter {
       
       @Override
       protected void configure(HttpSecurity http) throws Exception {
           http
               .authorizeRequests()
                   .antMatchers("/", "/login**").permitAll()
                   .anyRequest().authenticated()
                   .and()
               .oauth2Login()
                   .loginPage("/login")
                   .defaultSuccessUrl("/home")
                   .and()
               .logout()
                   .logoutSuccessUrl("/");
       }
   }
   ```

   ```properties
   # application.properties
   spring.security.oauth2.client.registration.google.client-id=${GOOGLE_CLIENT_ID}
   spring.security.oauth2.client.registration.google.client-secret=${GOOGLE_CLIENT_SECRET}
   spring.security.oauth2.client.registration.google.scope=openid,profile,email
   ```

6. **Database Schema for Authentication:**
   ```sql
   CREATE TABLE users (
       id BIGINT PRIMARY KEY AUTO_INCREMENT,
       username VARCHAR(50) NOT NULL UNIQUE,
       password VARCHAR(100) NOT NULL,
       enabled BOOLEAN NOT NULL
   );
   
   CREATE TABLE authorities (
       username VARCHAR(50) NOT NULL,
       authority VARCHAR(50) NOT NULL,
       CONSTRAINT fk_authorities_users FOREIGN KEY(username) REFERENCES users(username)
   );
   
   CREATE UNIQUE INDEX ix_auth_username ON authorities (username, authority);
   ```

7. **Remember-Me Authentication:**
   ```java
   @Override
   protected void configure(HttpSecurity http) throws Exception {
       http
           .authorizeRequests()
               // authorization rules
               .and()
           .formLogin()
               // login configuration
               .and()
           .rememberMe()
               .key("uniqueAndSecretKey")
               .tokenValiditySeconds(86400); // 1 day
   }
   ```

8. **Custom Authentication Success/Failure Handlers:**
   ```java
   @Component
   public class CustomAuthenticationSuccessHandler implements AuthenticationSuccessHandler {
       
       @Override
       public void onAuthenticationSuccess(HttpServletRequest request, 
                                          HttpServletResponse response, 
                                          Authentication authentication) 
                                          throws IOException, ServletException {
           // Custom logic after successful authentication
           String targetUrl = determineTargetUrl(authentication);
           response.sendRedirect(targetUrl);
       }
       
       private String determineTargetUrl(Authentication authentication) {
           boolean isUser = authentication.getAuthorities().stream()
               .anyMatch(a -> a.getAuthority().equals("ROLE_USER"));
               
           boolean isAdmin = authentication.getAuthorities().stream()
               .anyMatch(a -> a.getAuthority().equals("ROLE_ADMIN"));
               
           if (isAdmin) {
               return "/admin/dashboard";
           } else if (isUser) {
               return "/user/dashboard";
           } else {
               return "/home";
           }
       }
   }
   
   @Override
   protected void configure(HttpSecurity http) throws Exception {
       http
           .formLogin()
               .successHandler(authenticationSuccessHandler)
               .failureHandler(authenticationFailureHandler);
   }
   ```