# Spring MVC Interview Questions and Answers

## MVC Architecture

### What is the MVC architecture pattern?

**Answer:**
MVC (Model-View-Controller) is an architectural pattern that separates an application into three main components:

1. **Model:**
   - Represents the application data and business logic
   - Responsible for data access and manipulation
   - Independent of the user interface
   - Notifies observers (views) when data changes

2. **View:**
   - Represents the presentation layer
   - Renders the model data for the user
   - Sends user actions to the controller
   - Can be updated when the model changes

3. **Controller:**
   - Acts as an intermediary between Model and View
   - Processes incoming requests
   - Manipulates the model based on user input
   - Selects the appropriate view to render

**Benefits of MVC:**
- Separation of concerns
- Improved code organization and maintainability
- Parallel development (different teams can work on model, view, and controller)
- Easier testing and debugging
- Code reusability

### How does Spring implement the MVC pattern?

**Answer:**
Spring implements the MVC pattern through its Spring MVC framework:

1. **Model:**
   - Implemented using POJOs/beans
   - Managed through the Model interface or ModelAndView object
   - Can include service and repository layers for business logic and data access
   - Data can be added to the model using `model.addAttribute()`

2. **View:**
   - Implemented using various view technologies (JSP, Thymeleaf, FreeMarker, etc.)
   - Resolved by ViewResolvers
   - Renders the model data into HTML or other formats
   - Can be configured using view resolvers and template engines

3. **Controller:**
   - Implemented using classes annotated with `@Controller`
   - Handles HTTP requests using `@RequestMapping` methods
   - Processes input, updates the model, and returns the view name
   - Delegates business logic to service layer

**Spring MVC specific components:**
- **DispatcherServlet:** Front controller that dispatches requests to handlers
- **HandlerMapping:** Maps requests to handler methods
- **HandlerAdapter:** Helps DispatcherServlet invoke handler methods
- **ViewResolver:** Resolves view names to actual View implementations
- **ModelAndView:** Combines model and view information

**Example:**
```java
@Controller
public class UserController {
    
    private final UserService userService;
    
    public UserController(UserService userService) {
        this.userService = userService;
    }
    
    @GetMapping("/users")
    public String listUsers(Model model) {
        // Update model (M in MVC)
        model.addAttribute("users", userService.getAllUsers());
        
        // Return view name (V in MVC)
        return "user/list";
    }
}
```

## Request Flow

### Explain the flow in Spring MVC Framework

**Answer:**
The request flow in Spring MVC follows these steps:

1. **Client sends HTTP request:**
   - Browser or client application sends a request to the server

2. **DispatcherServlet receives the request:**
   - Front controller pattern implementation
   - Entry point for all requests
   - Configured in web.xml or through Java configuration

3. **HandlerMapping determines the controller:**
   - DispatcherServlet consults HandlerMapping
   - HandlerMapping identifies the appropriate controller and handler method
   - Returns a HandlerExecutionChain (handler + interceptors)

4. **DispatcherServlet invokes HandlerAdapter:**
   - HandlerAdapter executes the handler (controller method)
   - Converts request parameters to method arguments
   - Applies request interceptors

5. **Controller processes the request:**
   - Executes business logic
   - Updates the model with data
   - Returns a logical view name or ModelAndView object

6. **ViewResolver resolves the logical view name:**
   - Translates logical view name to actual View implementation
   - Determines which template to use

7. **View renders the response:**
   - View uses model data to render the response
   - Typically generates HTML, but can produce JSON, XML, PDF, etc.

8. **DispatcherServlet returns the response to the client:**
   - Rendered view is sent back to the client
   - Response includes HTTP status, headers, and body

**Diagram representation:**
```
Client → DispatcherServlet → HandlerMapping → HandlerAdapter → Controller → Service → Repository
                                                                    ↓
Client ← Response ← View ← ViewResolver ← ModelAndView
```

### What happens when a request is sent to a Spring MVC application?

**Answer:**
When a request is sent to a Spring MVC application, the following detailed sequence occurs:

1. **Request Reception:**
   - Client sends HTTP request to the server
   - Web container (e.g., Tomcat) receives the request
   - Request passes through filters (if configured)

2. **DispatcherServlet Processing:**
   - DispatcherServlet (front controller) receives the request
   - Creates or accesses the WebApplicationContext
   - Processes locale, theme, and multipart requests if needed

3. **Handler Resolution:**
   - DispatcherServlet consults registered HandlerMappings
   - HandlerMappings examine the request (URL, HTTP method, headers)
   - Appropriate handler (controller method) is selected
   - Handler execution chain is created with interceptors

4. **Handler Interceptor Pre-processing:**
   - `preHandle()` methods of interceptors are executed
   - Interceptors can modify request or short-circuit the execution

5. **Handler Execution:**
   - HandlerAdapter is selected based on handler type
   - Request parameters are bound to handler method arguments
   - Validation is performed if configured
   - Handler method is executed
   - Result is a ModelAndView object (explicit or implicit)

6. **Handler Interceptor Post-processing:**
   - `postHandle()` methods of interceptors are executed
   - Interceptors can modify the ModelAndView

7. **View Resolution:**
   - If view name is returned, ViewResolver resolves it to a View
   - If View is directly returned, resolution is skipped
   - Multiple ViewResolvers can be tried in order of priority

8. **View Rendering:**
   - View renders the output using the model data
   - Output format depends on view implementation (HTML, JSON, PDF, etc.)

9. **Handler Interceptor Completion:**
   - `afterCompletion()` methods of interceptors are executed
   - Used for cleanup operations

10. **Response Completion:**
    - Response is returned to the client
    - DispatcherServlet cleans up request attributes

**Example with timing:**
```
[10:15:00.100] Request received: GET /users
[10:15:00.105] DispatcherServlet processes request
[10:15:00.110] HandlerMapping finds UserController.listUsers()
[10:15:00.115] LoggingInterceptor.preHandle() executes
[10:15:00.120] UserController.listUsers() executes
[10:15:00.130] Service layer fetches data
[10:15:00.140] Model populated with user list
[10:15:00.145] LoggingInterceptor.postHandle() executes
[10:15:00.150] ViewResolver resolves "user/list" to UserListView
[10:15:00.160] UserListView renders HTML with model data
[10:15:00.165] LoggingInterceptor.afterCompletion() executes
[10:15:00.170] Response sent to client
```

### What are the key components involved in processing a request?

**Answer:**
The key components involved in processing a request in Spring MVC are:

1. **DispatcherServlet:**
   - Central component (front controller)
   - Coordinates the entire request processing workflow
   - Delegates to specialized components
   - Configured in web.xml or through `@EnableWebMvc`

2. **HandlerMapping:**
   - Maps requests to handlers (controller methods)
   - Different implementations available:
     - RequestMappingHandlerMapping (for @RequestMapping)
     - SimpleUrlHandlerMapping (for URL patterns)
     - BeanNameUrlHandlerMapping (matches URLs to bean names)
   - Returns HandlerExecutionChain with handler and interceptors

3. **HandlerAdapter:**
   - Adapts various types of handlers to a common interface
   - Invokes the actual handler method
   - Handles parameter binding and return value processing
   - Common implementations:
     - RequestMappingHandlerAdapter (for @RequestMapping methods)
     - HttpRequestHandlerAdapter (for HttpRequestHandlers)
     - SimpleControllerHandlerAdapter (for Controllers)

4. **Controller:**
   - Contains handler methods for processing requests
   - Annotated with @Controller
   - Methods annotated with @RequestMapping, @GetMapping, etc.
   - Updates model and returns view information

5. **HandlerInterceptor:**
   - Intercepts requests before and after handler execution
   - Can perform pre-processing and post-processing
   - Used for cross-cutting concerns (logging, security, etc.)
   - Methods: preHandle(), postHandle(), afterCompletion()

6. **HandlerExceptionResolver:**
   - Resolves exceptions thrown during request processing
   - Maps exceptions to views or other responses
   - Common implementations:
     - ExceptionHandlerExceptionResolver (for @ExceptionHandler)
     - DefaultHandlerExceptionResolver (for standard Spring MVC exceptions)
     - SimpleMappingExceptionResolver (for exception name to view mapping)

7. **ViewResolver:**
   - Resolves logical view names to actual View implementations
   - Multiple resolvers can be configured with different priorities
   - Common implementations:
     - InternalResourceViewResolver (for JSP)
     - ThymeleafViewResolver (for Thymeleaf)
     - FreeMarkerViewResolver (for FreeMarker)

8. **View:**
   - Renders the model data into the response
   - Different implementations for different view technologies
   - Handles content type, encoding, etc.

9. **LocaleResolver, ThemeResolver, MultipartResolver:**
   - Optional components for specific functionality
   - Handle internationalization, themes, and file uploads

**Configuration example:**
```java
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {
    
    @Bean
    public ViewResolver viewResolver() {
        InternalResourceViewResolver resolver = new InternalResourceViewResolver();
        resolver.setPrefix("/WEB-INF/views/");
        resolver.setSuffix(".jsp");
        return resolver;
    }
    
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new LoggingInterceptor());
    }
    
    @Bean
    public MultipartResolver multipartResolver() {
        return new StandardServletMultipartResolver();
    }
}
```

## View Resolution

### What is a ViewResolver in Spring MVC?

**Answer:**
A ViewResolver in Spring MVC is a component that translates the logical view names returned by controllers into actual View objects that render the response:

1. **Purpose:**
   - Maps logical view names to physical view resources
   - Decouples controllers from specific view technologies
   - Enables switching view technologies without changing controllers
   - Supports view caching for better performance

2. **Core functionality:**
   - Takes a String view name and Locale
   - Returns a View implementation
   - Implements the ViewResolver interface
   - Method: `View resolveViewName(String viewName, Locale locale)`

3. **Common ViewResolver implementations:**
   - **InternalResourceViewResolver:** For JSP views
   - **ThymeleafViewResolver:** For Thymeleaf templates
   - **FreeMarkerViewResolver:** For FreeMarker templates
   - **VelocityViewResolver:** For Velocity templates
   - **ContentNegotiatingViewResolver:** Selects view based on content type
   - **BeanNameViewResolver:** Maps view names to beans
   - **ResourceBundleViewResolver:** Maps view names from properties files
   - **XmlViewResolver:** Maps view names from XML files

4. **Chaining ViewResolvers:**
   - Multiple ViewResolvers can be configured
   - Each has an order property (lower values = higher priority)
   - Spring tries each resolver in order until one succeeds

**Example configuration:**
```java
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {
    
    @Bean
    public ViewResolver jspViewResolver() {
        InternalResourceViewResolver resolver = new InternalResourceViewResolver();
        resolver.setPrefix("/WEB-INF/views/");
        resolver.setSuffix(".jsp");
        resolver.setViewClass(JstlView.class);
        resolver.setOrder(2);
        return resolver;
    }
    
    @Bean
    public ViewResolver thymeleafViewResolver() {
        ThymeleafViewResolver resolver = new ThymeleafViewResolver();
        resolver.setTemplateEngine(templateEngine());
        resolver.setCharacterEncoding("UTF-8");
        resolver.setOrder(1);
        return resolver;
    }
    
    @Bean
    public ViewResolver beanNameViewResolver() {
        BeanNameViewResolver resolver = new BeanNameViewResolver();
        resolver.setOrder(0);
        return resolver;
    }
}
```

### What is the default ViewResolver in Spring MVC?

**Answer:**
Spring MVC doesn't have a single default ViewResolver that's automatically configured in all cases. The default behavior depends on the setup:

1. **In a basic Spring MVC setup:**
   - No ViewResolver is configured by default
   - You must explicitly configure at least one ViewResolver

2. **In Spring Boot:**
   - Spring Boot auto-configures ViewResolvers based on dependencies
   - If you have spring-boot-starter-thymeleaf, ThymeleafViewResolver is configured
   - If you have spring-boot-starter-freemarker, FreeMarkerViewResolver is configured
   - If you have spring-boot-starter-groovy-templates, GroovyMarkupViewResolver is configured
   - If none of these are present but JSP libraries are, InternalResourceViewResolver is configured

3. **Default fallback:**
   - If no ViewResolver is found, Spring MVC uses BeanNameViewResolver and InternalResourceViewResolver
   - InternalResourceViewResolver with prefix "/WEB-INF/" and suffix ".jsp"

4. **ContentNegotiatingViewResolver:**
   - In Spring Boot, this is configured automatically if needed
   - It's not a real view resolver but delegates to others based on content type

**Example of default Spring Boot configuration:**
```java
// This happens automatically in Spring Boot
@Configuration
public class WebMvcAutoConfiguration {
    
    @Bean
    @ConditionalOnMissingBean
    public InternalResourceViewResolver defaultViewResolver() {
        InternalResourceViewResolver resolver = new InternalResourceViewResolver();
        resolver.setPrefix("/WEB-INF/jsp/");
        resolver.setSuffix(".jsp");
        return resolver;
    }
    
    // Other view resolvers based on classpath dependencies
}
```

**How to check configured ViewResolvers:**
```java
@RestController
public class DebugController {
    
    @Autowired
    private List<ViewResolver> viewResolvers;
    
    @GetMapping("/debug/viewresolvers")
    public List<String> listViewResolvers() {
        return viewResolvers.stream()
            .map(vr -> vr.getClass().getName())
            .collect(Collectors.toList());
    }
}
```

### Can we use multiple view types in one project? How?

**Answer:**
Yes, Spring MVC supports using multiple view types (JSP, Thymeleaf, FreeMarker, etc.) in a single project. This can be achieved through several approaches:

1. **Configure multiple ViewResolvers:**
   - Define multiple ViewResolver beans with different priorities (order)
   - Spring tries each resolver in order until one succeeds
   - Each resolver handles a specific view technology

2. **Use ContentNegotiatingViewResolver:**
   - Acts as a master resolver that delegates to other resolvers
   - Selects view based on requested content type or extension
   - Useful for supporting multiple formats (HTML, JSON, PDF, etc.)

3. **Use BeanNameViewResolver:**
   - Maps view names directly to View beans
   - Different view beans can use different technologies
   - Useful for special views (PDF, Excel, etc.)

**Example configuration:**
```java
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {
    
    // For JSP views
    @Bean
    public ViewResolver jspViewResolver() {
        InternalResourceViewResolver resolver = new InternalResourceViewResolver();
        resolver.setPrefix("/WEB-INF/views/jsp/");
        resolver.setSuffix(".jsp");
        resolver.setViewClass(JstlView.class);
        resolver.setOrder(2);
        return resolver;
    }
    
    // For Thymeleaf views
    @Bean
    public ViewResolver thymeleafViewResolver() {
        ThymeleafViewResolver resolver = new ThymeleafViewResolver();
        resolver.setTemplateEngine(templateEngine());
        resolver.setCharacterEncoding("UTF-8");
        resolver.setViewNames(new String[] {"thymeleaf/*"});  // Only for views starting with "thymeleaf/"
        resolver.setOrder(1);
        return resolver;
    }
    
    // For FreeMarker views
    @Bean
    public ViewResolver freeMarkerViewResolver() {
        FreeMarkerViewResolver resolver = new FreeMarkerViewResolver();
        resolver.setPrefix("");
        resolver.setSuffix(".ftl");
        resolver.setViewNames(new String[] {"freemarker/*"});  // Only for views starting with "freemarker/"
        resolver.setOrder(3);
        return resolver;
    }
    
    // For special views (PDF, Excel, etc.)
    @Bean
    public ViewResolver beanNameViewResolver() {
        BeanNameViewResolver resolver = new BeanNameViewResolver();
        resolver.setOrder(0);  // Highest priority
        return resolver;
    }
    
    // PDF view bean
    @Bean
    public View pdfReport() {
        return new PdfReportView();
    }
    
    // Excel view bean
    @Bean
    public View excelReport() {
        return new ExcelReportView();
    }
    
    // Content negotiation
    @Bean
    public ViewResolver contentNegotiatingViewResolver(
            ContentNegotiationManager contentNegotiationManager,
            List<ViewResolver> viewResolvers) {
        
        ContentNegotiatingViewResolver resolver = new ContentNegotiatingViewResolver();
        resolver.setContentNegotiationManager(contentNegotiationManager);
        resolver.setViewResolvers(viewResolvers);
        resolver.setOrder(-1);  // Highest priority
        return resolver;
    }
}
```

**Controller using multiple view types:**
```java
@Controller
@RequestMapping("/users")
public class UserController {
    
    private final UserService userService;
    
    public UserController(UserService userService) {
        this.userService = userService;
    }
    
    // JSP view
    @GetMapping
    public String listUsersJsp(Model model) {
        model.addAttribute("users", userService.getAllUsers());
        return "user/list";  // Resolves to /WEB-INF/views/jsp/user/list.jsp
    }
    
    // Thymeleaf view
    @GetMapping("/thymeleaf")
    public String listUsersThymeleaf(Model model) {
        model.addAttribute("users", userService.getAllUsers());
        return "thymeleaf/user/list";  // Uses ThymeleafViewResolver
    }
    
    // FreeMarker view
    @GetMapping("/freemarker")
    public String listUsersFreemarker(Model model) {
        model.addAttribute("users", userService.getAllUsers());
        return "freemarker/user/list";  // Uses FreeMarkerViewResolver
    }
    
    // PDF view (using BeanNameViewResolver)
    @GetMapping("/pdf")
    public String getUsersPdf(Model model) {
        model.addAttribute("users", userService.getAllUsers());
        return "pdfReport";  // Maps to the pdfReport bean
    }
    
    // Content negotiation
    @GetMapping(path = "/{id}", produces = {"application/json", "application/xml", "text/html"})
    public User getUser(@PathVariable Long id) {
        return userService.getUserById(id);
        // View selected based on Accept header or extension (.json, .xml)
    }
}
```

## Controllers

### What is the role of a controller in Spring MVC?

**Answer:**
A controller in Spring MVC serves as the component that handles HTTP requests and defines the application's behavior. Its primary roles include:

1. **Request Handling:**
   - Receives HTTP requests from the DispatcherServlet
   - Maps URLs to specific handler methods
   - Extracts data from request parameters, path variables, headers, and body

2. **Input Processing:**
   - Validates input data
   - Converts request parameters to Java objects
   - Handles form submissions
   - Processes file uploads

3. **Business Logic Coordination:**
   - Delegates to service layer for business logic
   - Orchestrates the execution flow
   - Not responsible for implementing business logic directly

4. **Model Population:**
   - Creates and populates the Model with data
   - Prepares data to be rendered by the view
   - Passes attributes from services to views

5. **View Selection:**
   - Determines which view should render the response
   - Returns logical view names or ModelAndView objects
   - Can directly write to the response for REST APIs

6. **Exception Handling:**
   - Catches and processes exceptions
   - Maps exceptions to appropriate error views
   - Provides user-friendly error messages

7. **Response Management:**
   - Sets response status codes
   - Configures response headers
   - Controls caching behavior

**Example controller with various responsibilities:**
```java
@Controller
@RequestMapping("/products")
public class ProductController {
    
    private final ProductService productService;
    
    public ProductController(ProductService productService) {
        this.productService = productService;
    }
    
    // Basic request handling
    @GetMapping
    public String listProducts(Model model) {
        model.addAttribute("products", productService.getAllProducts());
        return "product/list";
    }
    
    // Path variable and model population
    @GetMapping("/{id}")
    public String viewProduct(@PathVariable Long id, Model model) {
        model.addAttribute("product", productService.getProductById(id));
        return "product/view";
    }
    
    // Form handling
    @GetMapping("/new")
    public String showProductForm(Model model) {
        model.addAttribute("product", new Product());
        return "product/form";
    }
    
    // Form submission and validation
    @PostMapping
    public String saveProduct(@Valid @ModelAttribute Product product, 
                             BindingResult result, 
                             RedirectAttributes redirectAttributes) {
        if (result.hasErrors()) {
            return "product/form";
        }
        
        productService.saveProduct(product);
        redirectAttributes.addFlashAttribute("message", "Product saved successfully!");
        return "redirect:/products";
    }
    
    // Exception handling
    @ExceptionHandler(ProductNotFoundException.class)
    public String handleProductNotFound(ProductNotFoundException ex, Model model) {
        model.addAttribute("errorMessage", ex.getMessage());
        return "error/not-found";
    }
    
    // REST endpoint
    @GetMapping(value = "/{id}", produces = "application/json")
    @ResponseBody
    public Product getProductJson(@PathVariable Long id) {
        return productService.getProductById(id);
    }
    
    // Common model attributes
    @ModelAttribute("categories")
    public List<Category> populateCategories() {
        return productService.getAllCategories();
    }
}
```

### What are the different ways to create a controller in Spring MVC?

**Answer:**
Spring MVC offers several approaches to create controllers, each with different characteristics and use cases:

1. **Annotation-based Controllers (@Controller):**
   - Most common and modern approach
   - Uses annotations to define mappings and behavior
   - Flexible method signatures
   - Support for various parameter types and return values

   ```java
   @Controller
   @RequestMapping("/users")
   public class UserController {
       
       @GetMapping
       public String listUsers(Model model) {
           // Implementation
           return "user/list";
       }
       
       @GetMapping("/{id}")
       public String viewUser(@PathVariable Long id, Model model) {
           // Implementation
           return "user/view";
       }
   }
   ```

2. **REST Controllers (@RestController):**
   - Specialized version of @Controller
   - Combines @Controller and @ResponseBody
   - Methods return domain objects instead of view names
   - Objects are automatically converted to JSON/XML

   ```java
   @RestController
   @RequestMapping("/api/users")
   public class UserRestController {
       
       @GetMapping
       public List<User> listUsers() {
           // Implementation
           return userService.getAllUsers();
       }
       
       @GetMapping("/{id}")
       public User getUser(@PathVariable Long id) {
           // Implementation
           return userService.getUserById(id);
       }
   }
   ```

3. **Controller Interface Implementation:**
   - Traditional approach (less common now)
   - Implements the Controller interface
   - Single handleRequest method handles all requests
   - Less flexible than annotation-based approach

   ```java
   public class OldStyleController implements Controller {
       
       @Override
       public ModelAndView handleRequest(HttpServletRequest request, 
                                        HttpServletResponse response) throws Exception {
           // Implementation
           ModelAndView mav = new ModelAndView("view-name");
           mav.addObject("message", "Hello World");
           return mav;
       }
   }
   ```

4. **AbstractController Extension:**
   - Extension of Controller interface
   - Provides common functionality
   - Still less flexible than annotation-based approach

   ```java
   public class UserAbstractController extends AbstractController {
       
       @Override
       protected ModelAndView handleRequestInternal(HttpServletRequest request, 
                                                  HttpServletResponse response) throws Exception {
           // Implementation
           return new ModelAndView("user/list", "users", userService.getAllUsers());
       }
   }
   ```

5. **HttpRequestHandler Implementation:**
   - Low-level request handling
   - Direct access to request and response
   - No automatic view resolution
   - Useful for specialized scenarios (file downloads, etc.)

   ```java
   public class FileDownloadController implements HttpRequestHandler {
       
       @Override
       public void handleRequest(HttpServletRequest request, 
                               HttpServletResponse response) throws ServletException, IOException {
           // Implementation
           response.setContentType("application/pdf");
           response.setHeader("Content-Disposition", "attachment; filename=report.pdf");
           // Write file content to response output stream
       }
   }
   ```

6. **Function-based Controllers (WebFlux):**
   - Used in reactive applications with Spring WebFlux
   - Functional programming style
   - Router functions define request mappings

   ```java
   @Configuration
   public class UserRouterConfig {
       
       @Bean
       public RouterFunction<ServerResponse> userRoutes(UserHandler userHandler) {
           return route()
               .GET("/users", userHandler::listUsers)
               .GET("/users/{id}", userHandler::getUser)
               .POST("/users", userHandler::createUser)
               .build();
       }
   }
   
   @Component
   public class UserHandler {
       
       public Mono<ServerResponse> listUsers(ServerRequest request) {
           // Implementation
           return ServerResponse.ok().body(userService.getAllUsers(), User.class);
       }
       
       public Mono<ServerResponse> getUser(ServerRequest request) {
           // Implementation
           String id = request.pathVariable("id");
           return ServerResponse.ok().body(userService.getUserById(id), User.class);
       }
   }
   ```

### How are request mappings defined in Spring MVC?

**Answer:**
Request mappings in Spring MVC define which controller methods handle specific HTTP requests. There are several ways to define these mappings:

1. **@RequestMapping Annotation:**
   - Most versatile mapping annotation
   - Can be applied at class and method levels
   - Supports multiple attributes for fine-grained mapping

   ```java
   @Controller
   @RequestMapping("/users")  // Base path for all methods
   public class UserController {
       
       @RequestMapping(method = RequestMethod.GET)  // Handles GET /users
       public String listUsers() {
           return "user/list";
       }
       
       @RequestMapping(value = "/{id}", method = RequestMethod.GET)  // Handles GET /users/{id}
       public String viewUser(@PathVariable Long id) {
           return "user/view";
       }
       
       @RequestMapping(method = RequestMethod.POST, consumes = "application/json")  // Handles POST /users with JSON
       public String createUser(@RequestBody User user) {
           return "redirect:/users";
       }
   }
   ```

2. **HTTP Method-Specific Annotations:**
   - Shorthand for common HTTP methods
   - More concise and readable
   - Same functionality as @RequestMapping with method attribute

   ```java
   @Controller
   @RequestMapping("/products")
   public class ProductController {
       
       @GetMapping  // Equivalent to @RequestMapping(method = RequestMethod.GET)
       public String listProducts() {
           return "product/list";
       }
       
       @PostMapping  // Equivalent to @RequestMapping(method = RequestMethod.POST)
       public String createProduct(@ModelAttribute Product product) {
           return "redirect:/products";
       }
       
       @PutMapping("/{id}")  // Equivalent to @RequestMapping(value = "/{id}", method = RequestMethod.PUT)
       public String updateProduct(@PathVariable Long id, @RequestBody Product product) {
           return "redirect:/products";
       }
       
       @DeleteMapping("/{id}")  // Equivalent to @RequestMapping(value = "/{id}", method = RequestMethod.DELETE)
       public String deleteProduct(@PathVariable Long id) {
           return "redirect:/products";
       }
       
       @PatchMapping("/{id}")  // Equivalent to @RequestMapping(value = "/{id}", method = RequestMethod.PATCH)
       public String partialUpdateProduct(@PathVariable Long id, @RequestBody Map<String, Object> updates) {
           return "redirect:/products";
       }
   }
   ```

3. **Request Mapping Attributes:**
   - Fine-tune request matching with various attributes
   - Can be combined for precise control

   ```java
   @Controller
   public class AdvancedMappingController {
       
       // Path variables
       @GetMapping("/users/{userId}/orders/{orderId}")
       public String getOrder(@PathVariable Long userId, @PathVariable Long orderId) {
           return "order/view";
       }
       
       // Request parameters
       @GetMapping(value = "/search", params = "type=user")
       public String searchUsers(@RequestParam String query) {
           return "search/users";
       }
       
       // Headers
       @GetMapping(value = "/api/data", headers = "API-Version=1")
       public String getDataV1() {
           return "api/v1/data";
       }
       
       // Content type (consumes)
       @PostMapping(value = "/api/users", consumes = "application/json")
       public String createUserJson(@RequestBody User user) {
           return "redirect:/users";
       }
       
       @PostMapping(value = "/api/users", consumes = "application/xml")
       public String createUserXml(@RequestBody User user) {
           return "redirect:/users";
       }
       
       // Accept header (produces)
       @GetMapping(value = "/api/users/{id}", produces = "application/json")
       @ResponseBody
       public User getUserJson(@PathVariable Long id) {
           return userService.getUserById(id);
       }
       
       @GetMapping(value = "/api/users/{id}", produces = "application/xml")
       @ResponseBody
       public User getUserXml(@PathVariable Long id) {
           return userService.getUserById(id);
       }
       
       // Regular expressions in paths
       @GetMapping("/files/{filename:.+}")
       public String getFile(@PathVariable String filename) {
           return "file/view";
       }
   }
   ```

4. **Functional Endpoints (WebFlux):**
   - Alternative to annotation-based mappings
   - Used in reactive applications
   - Programmatic definition of routes

   ```java
   @Configuration
   public class RouterConfig {
       
       @Bean
       public RouterFunction<ServerResponse> routes(UserHandler userHandler) {
           return route()
               .path("/users", builder -> builder
                   .GET("", userHandler::listUsers)
                   .POST("", userHandler::createUser)
                   .GET("/{id}", userHandler::getUser)
                   .PUT("/{id}", userHandler::updateUser)
                   .DELETE("/{id}", userHandler::deleteUser))
               .build();
       }
   }
   ```

5. **XML Configuration (Legacy):**
   - Older approach using XML instead of annotations
   - Rarely used in modern applications

   ```xml
   <beans>
       <bean id="userController" class="com.example.UserController"/>
       
       <bean class="org.springframework.web.servlet.handler.SimpleUrlHandlerMapping">
           <property name="mappings">
               <props>
                   <prop key="/users">userController</prop>
                   <prop key="/users/*">userController</prop>
               </props>
           </property>
       </bean>
   </beans>
   ```

## Model and ModelAndView

### What is the purpose of the Model in Spring MVC?

**Answer:**
The Model in Spring MVC serves as a container for data that will be used by the view to render the response. Its primary purposes include:

1. **Data Transfer:**
   - Acts as a bridge between controllers and views
   - Transfers data from business layer to presentation layer
   - Decouples data preparation from data presentation

2. **Attribute Storage:**
   - Stores key-value pairs (attributes)
   - Keys are strings, values can be any object
   - Attributes are accessible in the view template

3. **View Independence:**
   - Allows controllers to prepare data without knowing view details
   - Enables separation of concerns
   - Supports multiple view technologies

4. **Request Scope:**
   - Model attributes are available for the current request
   - Automatically added to request attributes
   - Accessible via ${attributeName} in view templates

**Model Interface:**
The Model is represented by the `org.springframework.ui.Model` interface with methods like:
- `Model addAttribute(String name, Object value)`
- `Model addAttribute(Object value)` (uses class name as attribute name)
- `Model addAllAttributes(Map<String, ?> attributes)`
- `boolean containsAttribute(String name)`

**Example usage:**
```java
@Controller
@RequestMapping("/products")
public class ProductController {
    
    private final ProductService productService;
    private final CategoryService categoryService;
    
    public ProductController(ProductService productService, CategoryService categoryService) {
        this.productService = productService;
        this.categoryService = categoryService;
    }
    
    @GetMapping
    public String listProducts(Model model) {
        // Add single attribute with explicit name
        model.addAttribute("products", productService.getAllProducts());
        
        // Add single attribute with default name (based on type)
        model.addAttribute(new SearchForm());  // Attribute name will be "searchForm"
        
        // Add multiple attributes
        Map<String, Object> attributes = new HashMap<>();
        attributes.put("categories", categoryService.getAllCategories());
        attributes.put("featuredProduct", productService.getFeaturedProduct());
        model.addAllAttributes(attributes);
        
        // Check if attribute exists
        if (!model.containsAttribute("message")) {
            model.addAttribute("message", "Welcome to our product catalog");
        }
        
        return "product/list";
    }
}
```

**In the view (JSP example):**
```jsp
<h1>Product Catalog</h1>
<p>${message}</p>

<form action="/products/search" method="get">
    <input type="text" name="query" value="${searchForm.query}">
    <button type="submit">Search</button>
</form>

<h2>Categories</h2>
<ul>
    <c:forEach items="${categories}" var="category">
        <li>${category.name}</li>
    </c:forEach>
</ul>

<h2>Featured Product</h2>
<div class="featured">
    <h3>${featuredProduct.name}</h3>
    <p>${featuredProduct.description}</p>
</div>

<h2>All Products</h2>
<ul>
    <c:forEach items="${products}" var="product">
        <li>${product.name} - $${product.price}</li>
    </c:forEach>
</ul>
```

### How is data passed from controllers to views?

**Answer:**
Spring MVC provides several mechanisms to pass data from controllers to views:

1. **Using Model Interface:**
   - Most common approach
   - Controller receives Model parameter
   - Attributes added to model are available in view

   ```java
   @GetMapping("/products")
   public String listProducts(Model model) {
       model.addAttribute("products", productService.getAllProducts());
       model.addAttribute("categories", categoryService.getAllCategories());
       return "product/list";
   }
   ```

2. **Using ModelMap:**
   - Similar to Model but extends LinkedHashMap
   - Provides additional map-like operations
   - Functionally equivalent to Model for basic use cases

   ```java
   @GetMapping("/products")
   public String listProducts(ModelMap modelMap) {
       modelMap.addAttribute("products", productService.getAllProducts());
       modelMap.put("categories", categoryService.getAllCategories());  // Map-style operation
       return "product/list";
   }
   ```

3. **Using ModelAndView:**
   - Combines model data and view name
   - Useful when view selection depends on controller logic
   - More explicit about both model and view

   ```java
   @GetMapping("/products")
   public ModelAndView listProducts() {
       ModelAndView mav = new ModelAndView("product/list");
       mav.addObject("products", productService.getAllProducts());
       mav.addObject("categories", categoryService.getAllCategories());
       return mav;
   }
   ```

4. **Using @ModelAttribute Method:**
   - Defines attributes available to all handler methods in controller
   - Executed before request handler methods
   - Useful for common data needed across multiple views

   ```java
   @Controller
   @RequestMapping("/products")
   public class ProductController {
       
       // This runs before any request handler method
       @ModelAttribute("categories")
       public List<Category> populateCategories() {
           return categoryService.getAllCategories();
       }
       
       @ModelAttribute("currentDate")
       public LocalDate getCurrentDate() {
           return LocalDate.now();
       }
       
       @GetMapping
       public String listProducts(Model model) {
           // "categories" and "currentDate" already in model
           model.addAttribute("products", productService.getAllProducts());
           return "product/list";
       }
   }
   ```

5. **Using @ModelAttribute Parameter:**
   - Binds request parameters to model object
   - Automatically adds object to model
   - Useful for form handling

   ```java
   @GetMapping("/products/search")
   public String searchProducts(@ModelAttribute SearchCriteria criteria, Model model) {
       // "criteria" is automatically added to model
       model.addAttribute("products", productService.searchProducts(criteria));
       return "product/search-results";
   }
   ```

6. **Using Map<String, Object> Parameter:**
   - Alternative to Model parameter
   - Attributes added to map are available in view

   ```java
   @GetMapping("/products")
   public String listProducts(Map<String, Object> model) {
       model.put("products", productService.getAllProducts());
       model.put("categories", categoryService.getAllCategories());
       return "product/list";
   }
   ```

7. **Using HttpServletRequest (Low-level):**
   - Direct access to request attributes
   - Less common in modern Spring MVC applications

   ```java
   @GetMapping("/products")
   public String listProducts(HttpServletRequest request) {
       request.setAttribute("products", productService.getAllProducts());
       request.setAttribute("categories", categoryService.getAllCategories());
       return "product/list";
   }
   ```

8. **Using RedirectAttributes (for redirects):**
   - Passes data across a redirect
   - FlashAttributes survive redirect and are removed after next request
   - Useful for success/error messages after form submission

   ```java
   @PostMapping("/products")
   public String createProduct(@ModelAttribute Product product, RedirectAttributes redirectAttributes) {
       productService.saveProduct(product);
       redirectAttributes.addFlashAttribute("message", "Product created successfully!");
       redirectAttributes.addAttribute("category", product.getCategory().getId());  // Added to URL
       return "redirect:/products";
   }
   ```

### What is ModelAndView and when would you use it?

**Answer:**
ModelAndView is a holder object that combines model data and view information in a single return value. It encapsulates both the logical view name and the model attributes that should be displayed in that view.

**Key characteristics:**
1. Combines model (data) and view (template) information
2. Can be instantiated with or without a view name
3. Allows adding model attributes after creation
4. Can be used to set view object directly instead of view name
5. Supports redirect view and other special views

**When to use ModelAndView:**

1. **When view selection is conditional:**
   - When the view depends on business logic or request parameters
   - When different views might be selected based on conditions

   ```java
   @GetMapping("/report")
   public ModelAndView generateReport(@RequestParam String format) {
       List<DataPoint> data = reportService.generateReportData();
       
       ModelAndView mav = new ModelAndView();
       mav.addObject("reportData", data);
       mav.addObject("generatedDate", LocalDate.now());
       
       if ("pdf".equals(format)) {
           mav.setViewName("pdfReportView");
       } else if ("excel".equals(format)) {
           mav.setViewName("excelReportView");
       } else {
           mav.setViewName("report/standard");
       }
       
       return mav;
   }
   ```

2. **When using programmatically created views:**
   - When using custom View implementations
   - When view object needs to be configured programmatically

   ```java
   @GetMapping("/users/export")
   public ModelAndView exportUsers() {
       List<User> users = userService.getAllUsers();
       
       // Create custom view object
       CsvView csvView = new CsvView();
       csvView.setFileName("users.csv");
       csvView.setColumns(new String[]{"id", "username", "email"});
       
       return new ModelAndView(csvView, "users", users);
   }
   ```

3. **When using special view types:**
   - When using redirect views
   - When using forward views
   - When using special view resolvers

   ```java
   @PostMapping("/orders")
   public ModelAndView createOrder(@ModelAttribute Order order) {
       orderService.placeOrder(order);
       
       // Using redirect view
       RedirectView redirectView = new RedirectView("/orders/confirmation");
       redirectView.setExposeModelAttributes(false);
       
       ModelAndView mav = new ModelAndView(redirectView);
       mav.addObject("orderId", order.getId());
       return mav;
   }
   ```

4. **When working with legacy code:**
   - When extending AbstractController or other legacy base classes
   - When implementing Controller interface

   ```java
   public class LegacyController extends AbstractController {
       
       @Override
       protected ModelAndView handleRequestInternal(HttpServletRequest request, 
                                                  HttpServletResponse response) {
           // Business logic
           List<Product> products = productService.getAllProducts();
           
           ModelAndView mav = new ModelAndView("product/list");
           mav.addObject("products", products);
           return mav;
       }
   }
   ```

5. **When you prefer explicit style:**
   - When you want to make it clear that both model and view are being managed
   - When you find it more readable to group model and view operations

   ```java
   @GetMapping("/dashboard")
   public ModelAndView showDashboard() {
       ModelAndView mav = new ModelAndView("dashboard/main");
       mav.addObject("stats", statsService.getSummaryStats());
       mav.addObject("recentActivity", activityService.getRecentActivity());
       mav.addObject("notifications", notificationService.getPendingNotifications());
       return mav;
   }
   ```

**ModelAndView vs. Model + View Name:**
Both approaches are functionally equivalent in most cases:

```java
// Using Model + view name
@GetMapping("/products")
public String listProducts(Model model) {
    model.addAttribute("products", productService.getAllProducts());
    return "product/list";
}

// Using ModelAndView
@GetMapping("/products")
public ModelAndView listProducts() {
    ModelAndView mav = new ModelAndView("product/list");
    mav.addObject("products", productService.getAllProducts());
    return mav;
}
```

The choice often comes down to personal or team preference, unless you need the specific capabilities of ModelAndView mentioned above.

## Form Handling

### How are forms handled in Spring MVC?

**Answer:**
Spring MVC provides a comprehensive framework for handling HTML forms, with support for data binding, validation, and error handling. The form handling process typically involves these steps:

1. **Displaying the Form:**
   - Controller method prepares and returns a form view
   - Empty or pre-populated form model object is added to the model
   - View renders HTML form with form tags

2. **Form Submission:**
   - User fills out and submits the form
   - Request is sent to the form processing URL
   - Controller binds form data to a model object
   - Validation is performed
   - Based on validation results, success view or form view with errors is shown

**Complete form handling example:**

1. **Model class:**
```java
public class UserForm {
    private String username;
    private String email;
    private String password;
    private boolean receiveNewsletter;
    
    // Getters and setters
}
```

2. **Controller with form handling methods:**
```java
@Controller
@RequestMapping("/users")
public class UserController {
    
    private final UserService userService;
    
    public UserController(UserService userService) {
        this.userService = userService;
    }
    
    // Display the form
    @GetMapping("/register")
    public String showRegistrationForm(Model model) {
        // Add empty form object to the model
        model.addAttribute("userForm", new UserForm());
        return "user/register";
    }
    
    // Process form submission
    @PostMapping("/register")
    public String processRegistration(
            @Valid @ModelAttribute("userForm") UserForm userForm,
            BindingResult result,
            RedirectAttributes redirectAttributes) {
        
        // Custom validation
        if (userService.isUsernameTaken(userForm.getUsername())) {
            result.rejectValue("username", "duplicate", "Username already taken");
        }
        
        // If validation errors exist, return to form
        if (result.hasErrors()) {
            return "user/register";
        }
        
        // Process valid form data
        userService.registerUser(userForm);
        
        // Add flash message for next request
        redirectAttributes.addFlashAttribute("message", "Registration successful!");
        
        // Redirect to prevent duplicate submission
        return "redirect:/users/login";
    }
}
```

3. **JSP form view (using Spring form tags):**
```jsp
<%@ taglib prefix="form" uri="http://www.springframework.org/tags/form" %>
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>

<h1>User Registration</h1>

<c:if test="${not empty message}">
    <div class="alert">${message}</div>
</c:if>

<form:form modelAttribute="userForm" method="post">
    <div>
        <form:label path="username">Username</form:label>
        <form:input path="username" />
        <form:errors path="username" cssClass="error" />
    </div>
    
    <div>
        <form:label path="email">Email</form:label>
        <form:input path="email" type="email" />
        <form:errors path="email" cssClass="error" />
    </div>
    
    <div>
        <form:label path="password">Password</form:label>
        <form:password path="password" />
        <form:errors path="password" cssClass="error" />
    </div>
    
    <div>
        <form:checkbox path="receiveNewsletter" />
        <form:label path="receiveNewsletter">Subscribe to newsletter</form:label>
    </div>
    
    <div>
        <button type="submit">Register</button>
    </div>
</form:form>
```

4. **Thymeleaf form view alternative:**
```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <title>User Registration</title>
</head>
<body>
    <h1>User Registration</h1>
    
    <div th:if="${message}" class="alert" th:text="${message}"></div>
    
    <form th:action="@{/users/register}" th:object="${userForm}" method="post">
        <div>
            <label for="username">Username</label>
            <input type="text" th:field="*{username}" />
            <span th:if="${#fields.hasErrors('username')}" th:errors="*{username}" class="error"></span>
        </div>
        
        <div>
            <label for="email">Email</label>
            <input type="email" th:field="*{email}" />
            <span th:if="${#fields.hasErrors('email')}" th:errors="*{email}" class="error"></span>
        </div>
        
        <div>
            <label for="password">Password</label>
            <input type="password" th:field="*{password}" />
            <span th:if="${#fields.hasErrors('password')}" th:errors="*{password}" class="error"></span>
        </div>
        
        <div>
            <input type="checkbox" th:field="*{receiveNewsletter}" />
            <label for="receiveNewsletter">Subscribe to newsletter</label>
        </div>
        
        <div>
            <button type="submit">Register</button>
        </div>
    </form>
</body>
</html>
```

**Additional form handling features:**

1. **Pre-populating forms for editing:**
```java
@GetMapping("/users/{id}/edit")
public String showEditForm(@PathVariable Long id, Model model) {
    User user = userService.getUserById(id);
    UserForm userForm = convertToForm(user);
    model.addAttribute("userForm", userForm);
    return "user/edit";
}
```

2. **File uploads:**
```java
@PostMapping("/upload")
public String handleFileUpload(@RequestParam("file") MultipartFile file, RedirectAttributes redirectAttributes) {
    if (file.isEmpty()) {
        redirectAttributes.addFlashAttribute("error", "Please select a file");
        return "redirect:/upload";
    }
    
    try {
        storageService.store(file);
        redirectAttributes.addFlashAttribute("message", "File uploaded successfully");
    } catch (Exception e) {
        redirectAttributes.addFlashAttribute("error", "Failed to upload file");
    }
    
    return "redirect:/upload";
}
```

3. **Multiple submit buttons:**
```html
<form th:action="@{/orders}" th:object="${order}" method="post">
    <!-- Form fields -->
    
    <button type="submit" name="action" value="save">Save Draft</button>
    <button type="submit" name="action" value="submit">Submit Order</button>
</form>
```

```java
@PostMapping("/orders")
public String processOrder(@ModelAttribute Order order, @RequestParam String action) {
    if ("save".equals(action)) {
        orderService.saveDraft(order);
        return "redirect:/orders/drafts";
    } else if ("submit".equals(action)) {
        orderService.submitOrder(order);
        return "redirect:/orders/confirmation";
    }
    return "redirect:/orders";
}
```

4. **Dynamic form fields:**
```html
<form th:action="@{/products}" th:object="${product}" method="post">
    <input type="text" th:field="*{name}" />
    
    <div th:each="attribute, status : *{attributes}">
        <input type="text" th:field="*{attributes[__${status.index}__].name}" />
        <input type="text" th:field="*{attributes[__${status.index}__].value}" />
    </div>
    
    <button type="button" onclick="addAttribute()">Add Attribute</button>
    <button type="submit">Save Product</button>
</form>
```

### What is form binding and validation in Spring MVC?

**Answer:**
Form binding and validation are core features of Spring MVC that simplify working with HTML forms and ensure data integrity.

**Form Binding:**
Form binding is the process of mapping HTML form fields to Java objects and vice versa. It involves:

1. **Data Binding:**
   - Converting HTTP request parameters to Java object properties
   - Handling type conversion (strings to numbers, dates, etc.)
   - Populating form objects from request data

2. **Binding Process:**
   - Spring uses DataBinder internally
   - PropertyEditors or Converter/Formatter for type conversion
   - Supports nested properties and collections
   - Handles both simple types and complex objects

3. **Key Components:**
   - @ModelAttribute: Binds form data to model objects
   - BindingResult: Holds binding and validation errors
   - WebDataBinder: Configures binding behavior

**Form Validation:**
Validation ensures that submitted data meets business rules and constraints. Spring MVC supports:

1. **Bean Validation (JSR-380):**
   - Annotation-based validation (@NotNull, @Size, etc.)
   - Standard validation API integrated with Spring
   - Implemented by Hibernate Validator by default

2. **Spring Validation:**
   - Validator interface for custom validation
   - ValidationUtils helper class
   - Integrated with data binding

3. **Validation Process:**
   - Annotations define constraints on model properties
   - Spring validates object after binding
   - Errors stored in BindingResult
   - Controller checks for errors and responds accordingly

**Complete Example:**

1. **Model with validation annotations:**
```java
public class RegistrationForm {
    
    @NotBlank(message = "Username is required")
    @Size(min = 4, max = 20, message = "Username must be between 4 and 20 characters")
    @Pattern(regexp = "^[a-zA-Z0-9_]+$", message = "Username can only contain letters, numbers and underscore")
    private String username;
    
    @NotBlank(message = "Email is required")
    @Email(message = "Please provide a valid email address")
    private String email;
    
    @NotBlank(message = "Password is required")
    @Size(min = 8, message = "Password must be at least 8 characters long")
    private String password;
    
    @NotBlank(message = "Password confirmation is required")
    private String confirmPassword;
    
    @AssertTrue(message = "You must agree to the terms")
    private boolean termsAccepted;
    
    // Getters and setters
}
```

2. **Custom validator for cross-field validation:**
```java
@Component
public class RegistrationValidator implements Validator {
    
    @Override
    public boolean supports(Class<?> clazz) {
        return RegistrationForm.class.equals(clazz);
    }
    
    @Override
    public void validate(Object target, Errors errors) {
        RegistrationForm form = (RegistrationForm) target;
        
        // Cross-field validation
        if (form.getPassword() != null && form.getConfirmPassword() != null &&
                !form.getPassword().equals(form.getConfirmPassword())) {
            errors.rejectValue("confirmPassword", "password.mismatch", 
                              "Password confirmation doesn't match password");
        }
    }
}
```

3. **Controller with binding and validation:**
```java
@Controller
@RequestMapping("/register")
public class RegistrationController {
    
    private final UserService userService;
    private final RegistrationValidator registrationValidator;
    
    public RegistrationController(UserService userService, RegistrationValidator registrationValidator) {
        this.userService = userService;
        this.registrationValidator = registrationValidator;
    }
    
    @InitBinder("registrationForm")
    public void initBinder(WebDataBinder binder) {
        // Register custom validator
        binder.addValidators(registrationValidator);
        
        // Custom property editor for date fields
        binder.registerCustomEditor(Date.class, new CustomDateEditor(
            new SimpleDateFormat("yyyy-MM-dd"), true));
            
        // Disallow certain fields (for security)
        binder.setDisallowedFields("admin", "role");
    }
    
    @GetMapping
    public String showRegistrationForm(Model model) {
        model.addAttribute("registrationForm", new RegistrationForm());
        return "register";
    }
    
    @PostMapping
    public String processRegistration(
            @Valid @ModelAttribute("registrationForm") RegistrationForm form,
            BindingResult result,
            RedirectAttributes redirectAttributes) {
        
        // Check for validation errors from annotations and custom validator
        if (result.hasErrors()) {
            return "register";
        }
        
        // Business logic validation
        if (userService.isUsernameTaken(form.getUsername())) {
            result.rejectValue("username", "duplicate", "Username already taken");
            return "register";
        }
        
        // Process valid form
        userService.registerUser(form);
        redirectAttributes.addFlashAttribute("message", "Registration successful!");
        return "redirect:/login";
    }
}
```

4. **Thymeleaf template with error display:**
```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <title>Registration</title>
    <style>
        .error { color: red; }
        .field-error { border-color: red; }
    </style>
</head>
<body>
    <h1>Registration Form</h1>
    
    <div th:if="${message}" th:text="${message}"></div>
    
    <form th:action="@{/register}" th:object="${registrationForm}" method="post">
        <!-- Global errors -->
        <div th:if="${#fields.hasGlobalErrors()}">
            <p th:each="err : ${#fields.globalErrors()}" th:text="${err}" class="error"></p>
        </div>
        
        <div>
            <label for="username">Username</label>
            <input type="text" th:field="*{username}" th:errorclass="field-error" />
            <span th:if="${#fields.hasErrors('username')}" th:errors="*{username}" class="error"></span>
        </div>
        
        <div>
            <label for="email">Email</label>
            <input type="email" th:field="*{email}" th:errorclass="field-error" />
            <span th:if="${#fields.hasErrors('email')}" th:errors="*{email}" class="error"></span>
        </div>
        
        <div>
            <label for="password">Password</label>
            <input type="password" th:field="*{password}" th:errorclass="field-error" />
            <span th:if="${#fields.hasErrors('password')}" th:errors="*{password}" class="error"></span>
        </div>
        
        <div>
            <label for="confirmPassword">Confirm Password</label>
            <input type="password" th:field="*{confirmPassword}" th:errorclass="field-error" />
            <span th:if="${#fields.hasErrors('confirmPassword')}" th:errors="*{confirmPassword}" class="error"></span>
        </div>
        
        <div>
            <input type="checkbox" th:field="*{termsAccepted}" th:errorclass="field-error" />
            <label for="termsAccepted">I accept the terms and conditions</label>
            <span th:if="${#fields.hasErrors('termsAccepted')}" th:errors="*{termsAccepted}" class="error"></span>
        </div>
        
        <div>
            <button type="submit">Register</button>
        </div>
    </form>
</body>
</html>
```

**Advanced Form Binding and Validation Features:**

1. **Custom validation annotations:**
```java
@Target({ElementType.FIELD, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Constraint(validatedBy = UniqueUsernameValidator.class)
public @interface UniqueUsername {
    String message() default "Username already exists";
    Class<?>[] groups() default {};
    Class<? extends Payload>[] payload() default {};
}

public class UniqueUsernameValidator implements ConstraintValidator<UniqueUsername, String> {
    
    private final UserRepository userRepository;
    
    public UniqueUsernameValidator(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
    
    @Override
    public boolean isValid(String username, ConstraintValidatorContext context) {
        return username != null && !userRepository.existsByUsername(username);
    }
}
```

2. **Validation groups for different scenarios:**
```java
// Validation groups
public interface ValidationGroups {
    interface Create {}
    interface Update {}
}

public class UserDto {
    
    @NotNull(groups = ValidationGroups.Update.class)
    private Long id;
    
    @NotBlank(groups = {ValidationGroups.Create.class, ValidationGroups.Update.class})
    @UniqueUsername(groups = ValidationGroups.Create.class)
    private String username;
    
    // Other fields
}

@PostMapping
public String createUser(
        @Validated(ValidationGroups.Create.class) @ModelAttribute UserDto userDto,
        BindingResult result) {
    // Validation uses only Create group constraints
    if (result.hasErrors()) {
        return "user/form";
    }
    // Process form
}
```

3. **Programmatic validation:**
```java
@PostMapping("/users")
public String createUser(@ModelAttribute UserDto userDto, BindingResult result) {
    // Manual validation
    ValidationUtils.rejectIfEmptyOrWhitespace(result, "username", "field.required", "Username is required");
    
    if (userDto.getUsername() != null && userDto.getUsername().length() < 4) {
        result.rejectValue("username", "field.min.length", 
                          new Object[]{4}, "Username must be at least 4 characters");
    }
    
    // Check result
    if (result.hasErrors()) {
        return "user/form";
    }
    
    // Process form
    return "redirect:/users";
}
```