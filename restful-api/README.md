# RESTful API Interview Questions

## REST Fundamentals

- What is REST?
- What are the key principles of REST?
- What are the constraints of REST architecture?

## API Design

- What are the best practices for RESTful API design?
- How to design resource URIs?
- How to handle versioning in RESTful APIs?

## HTTP Methods

- What is the difference between POST and GET?
- What is the difference between PUT, POST, and PATCH?
- What is the meaning of partial operation in REST?
- What happens if two methods are called on the same resource?

## Advantages and Disadvantages

- What are the advantages of RESTful Web Services?
- What are the disadvantages of RESTful Web Services?

## API Documentation

- How to implement Swagger or OpenAPI for creating documentation of APIs?
- What are the benefits of using Swagger/OpenAPI?

## REST Clients

- What is RestTemplate and how is it used?
- What is WebClient and how is it different from RestTemplate?
- Is WebClient asynchronous?

## JAX-RS

- What is the JAX-RS library?
- How does serialization happen in REST? (converting object to JSON and JSON to object)

## Code Examples

```java
// Example: RESTful Controller with different HTTP methods
@RestController
@RequestMapping("/api/products")
public class ProductController {
    
    private final ProductService productService;
    
    public ProductController(ProductService productService) {
        this.productService = productService;
    }
    
    @GetMapping
    public List<Product> getAllProducts() {
        return productService.findAll();
    }
    
    @GetMapping("/{id}")
    public ResponseEntity<Product> getProductById(@PathVariable Long id) {
        return productService.findById(id)
            .map(ResponseEntity::ok)
            .orElse(ResponseEntity.notFound().build());
    }
    
    @PostMapping
    public ResponseEntity<Product> createProduct(@RequestBody @Valid Product product) {
        Product savedProduct = productService.save(product);
        URI location = ServletUriComponentsBuilder.fromCurrentRequest()
            .path("/{id}")
            .buildAndExpand(savedProduct.getId())
            .toUri();
        return ResponseEntity.created(location).body(savedProduct);
    }
    
    @PutMapping("/{id}")
    public ResponseEntity<Product> updateProduct(@PathVariable Long id, @RequestBody @Valid Product product) {
        return productService.update(id, product)
            .map(ResponseEntity::ok)
            .orElse(ResponseEntity.notFound().build());
    }
    
    @PatchMapping("/{id}")
    public ResponseEntity<Product> partialUpdateProduct(@PathVariable Long id, @RequestBody Map<String, Object> updates) {
        return productService.partialUpdate(id, updates)
            .map(ResponseEntity::ok)
            .orElse(ResponseEntity.notFound().build());
    }
    
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteProduct(@PathVariable Long id) {
        if (productService.deleteById(id)) {
            return ResponseEntity.noContent().build();
        }
        return ResponseEntity.notFound().build();
    }
}
```

## Resources

- [RESTful API Resources](https://docs.google.com/document/d/12fm5rtLH1rsYOeVMNM2OWaAQj4nN_SXwYh6bsWVvXtg/edit)
- [Additional RESTful API Resources](https://drive.google.com/drive/u/1/folders/1IP1eFO10BkqVoe9tXo97YM4UwJ_d09OZ)