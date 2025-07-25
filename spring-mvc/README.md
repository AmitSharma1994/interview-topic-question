# Spring MVC Interview Questions

## MVC Architecture

- What is the MVC architecture pattern?
- How does Spring implement the MVC pattern?

## Request Flow

- Explain the flow in Spring MVC Framework
- What happens when a request is sent to a Spring MVC application?
- What are the key components involved in processing a request?

## View Resolution

- What is a ViewResolver in Spring MVC?
- What is the default ViewResolver in Spring MVC?
- Can we use multiple view types in one project? How?

## Controllers

- What is the role of a controller in Spring MVC?
- What are the different ways to create a controller in Spring MVC?
- How are request mappings defined in Spring MVC?

## Model and ModelAndView

- What is the purpose of the Model in Spring MVC?
- How is data passed from controllers to views?
- What is ModelAndView and when would you use it?

## Form Handling

- How are forms handled in Spring MVC?
- What is form binding and validation in Spring MVC?

## Code Examples

```java
// Example: Basic Spring MVC Controller
@Controller
@RequestMapping("/users")
public class UserController {
    
    private final UserService userService;
    
    public UserController(UserService userService) {
        this.userService = userService;
    }
    
    @GetMapping
    public String listUsers(Model model) {
        model.addAttribute("users", userService.getAllUsers());
        return "user/list";
    }
    
    @GetMapping("/{id}")
    public String viewUser(@PathVariable Long id, Model model) {
        model.addAttribute("user", userService.getUserById(id));
        return "user/view";
    }
}
```

## Resources

- [Spring MVC Resources](https://docs.google.com/document/d/1YHOAsOcJISObTusApb7E3LjO5ASu6UfFB3SqrGJzhmM/edit)