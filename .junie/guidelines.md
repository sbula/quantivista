# Spring Boot Guidelines

## Project Structure
* Follow package-by-feature/module and in each module package-by-layer code organization style:
````
project-root/
├── pom.xml
├── src/
│   ├── main/
│   │   ├── java/
│   │   │   └── com/mycompany/projectname/
│   │   │       ├── config/
│   │   │       ├── module1/
│   │   │       │     ├── api/
│   │   │       │     │   ├── controller/
│   │   │       │     │   └── dto/
│   │   │       │     ├── config/
│   │   │       │     ├── domain/
│   │   │       │     │   ├── repository/
│   │   │       │     │   ├── exception/
│   │   │       │     │   ├── mapper/
│   │   │       │     │   ├── model/
│   │   │       │     │   └── service/
│   │   │       │     ├── job/
│   │   │       │     ├── eventhandler/
│   │   ├── proto/
│   │   └── resources/
│   │       └── static/
│   │       └── templates/
│   │       └── application.yaml
│   └── test/
│   │   └── java/
│   │   │   ├── com/mycompany/projectname/
│   │   │   │   ├── module1/
│   │   │   │   │     ├── api/
│   │   │   │   │     │   ├── controller/
│   │   │   │   │     ├── domain/
│   │   │   │   │     │   ├── repository/
│   │   │   │   │     │   ├── mapper/
│   │   │   │   │     │   ├── model/
│   │   │   │   │     │   └── service/
│   │   │   │   ├── helper/
│   │   │   │   └── integration/
│   │   │   │   │     ├── config/
│   │   │   │   │     ├── controller/
│   │   │   │   │     ├── testcontainer/
├── compose.yaml
└── README.md
````
## 1. Web Layer (com.companyname.projectname.module.api):
* Controllers handle HTTP requests and responses
* DTOs for request/response data
* Global exception handling

## 2. Service Layer (com.companyname.projectname.module.domain.service):
* Business logic implementation
* Transaction management

## 3. Entity Layer (com.companyname.projectname.module.domain.entity):
* JPA entities representing database tables

## 4. Model Layer (com.companyname.projectname.module.domain.model):
* DTOs for domain objects
* Command objects for operations

## 5. Mapper Layer (com.companyname.projectname.module.domain.mapper):
* Converters from DTOs to JPA entities and vice-versa

## 6. Exceptions (com.companyname.projectname.module.domain.exception):
* Custom exceptions

## 7. Config (com.companyname.projectname.module.config):
* Spring Boot configuration classes such as WebMvcConfig, WebSecurityConfig, etc.

## Java Code Style Guidelines

## 1. Java Code Style:
* Use Java 21 features where appropriate (records, text blocks, pattern matching, etc.)
* Follow standard Java naming conventions
* Use meaningful variable and method names
* Use public access modifier only when necessary

## 2. Test Style:
* Use descriptive test method names
* Follow the Given-When-Then pattern
* Use AssertJ for assertions
* Use Testcontainers for integration tests
  * Spin up real services (databases, message brokers, etc.) in your integration tests to mirror production environments.
  * Use RESTassured for integration tests
  * Separate integration test from unit test, so they can be executed easily and independently of each other.
  * Do NOT use a single mock in integration tests
  * Use random port for integration tests
    * When writing integration tests, start the application on a random available port to avoid port conflicts by annotating the test class with:

    ```java
    @SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
    ```

## Spring Boot Code Style Guidelines

## 1. Dependency Injection Style
* Don't use Spring Field Injection in production code.
* Use Constructor Injection without adding @Autowired.
* Declare all the mandatory dependencies as `final` fields and inject them through the constructor.

## 2. Clear Transactional Boundaries
* Make a business logic layer (@Service classes) as a transactional boundary.
* Annotate methods that perform DB read-only operations with @Transactional(readOnly=true).
* Annotate methods that perform DB write operations with @Transactional.
* Limit the code inside each transaction to the smallest necessary scope.

## 3. Separate Web Layer from Persistence Layer
* Don't expose entities directly as responses in controllers.
* Define explicit request and response record (DTO) classes instead.
* Apply Jakarta Validation annotations on your request records to enforce input rules.

## 4. Create custom Spring Data JPA methods with meaningful method names using JPQL queries instead of using long derived query method names.

## 5. Create usecase specific Command objects and pass them to the "service" layer methods to perform create or update operations.

## 6. Application Configuration:
* Create all the application-specific configuration properties with a common prefix in application.yml file.
* Use Typed Configuration with @ConfigurationProperties with validations.
* Organize Configuration with Typed Properties
  * Group application-specific configuration properties with a common prefix in `application.properties` or `.yml`.
  * Bind them to `@ConfigurationProperties` classes with validation annotations so that the application will fail fast if the configuration is invalid.
  * Prefer environment variables instead of profiles for passing different configuration properties for different environments.

## 7. Centralize Exception Handling
* Define a global handler class annotated with `@ControllerAdvice` (or `@RestControllerAdvice` for REST APIs) using `@ExceptionHandler` methods to handle specific exceptions.
* Return consistent error responses. Consider using the ProblemDetails response format ([RFC 9457](https://www.rfc-editor.org/rfc/rfc9457)).

## 8. Logging:
* **Use a proper logging framework.**  
  Never use `System.out.println()` for application logging. Rely on SLF4J (or a compatible abstraction) and your chosen backend (Logback, Log4j2, etc.).
* **Protect sensitive data.**  
  Ensure that no credentials, personal information, or other confidential details ever appear in log output.
* **Guard expensive log calls.**  
  When building verbose messages at `DEBUG` or `TRACE` level, especially those involving method calls or complex string concatenations, wrap them in a level check or use suppliers:

```java
if (logger.isDebugEnabled()) {
    logger.debug("Detailed state: {}", computeExpensiveDetails());
}

// using Supplier/Lambda expression
logger.atDebug()
	.setMessage("Detailed state: {}")
	.addArgument(() -> computeExpensiveDetails())
    .log();
```


## 9. Use WebJars for service static content.


## 10. Actuator
* Expose only essential actuator endpoints (such as `/health`, `/info`, `/metrics`) without requiring authentication. All the other actuator endpoints must be secured.


## 11. Follow REST API Design Principles
* **Versioned, resource-oriented URLs:** Structure your endpoints as `/api/v{version}/resources` (e.g. `/api/v1/orders`).
* **Consistent patterns for collections and sub-resources:** Keep URL conventions uniform (for example, `/posts` for posts collection and `/posts/{slug}/comments` for comments of a specific post).
* **Explicit HTTP status codes via ResponseEntity:** Use `ResponseEntity<T>` to return the correct status (e.g. 200 OK, 201 Created, 404 Not Found) along with the response body.
* Use pagination for collection resources that may contain an unbounded number of items.
* The JSON payload must use a JSON object as a top-level data structure to allow for future extension.
* Use snake_case or camelCase for JSON property names consistently.
* **Document APIs with OpenAPI annotations:** Always document REST APIs using OpenAPI annotations to ensure comprehensive API documentation:
  * Use `@Operation` annotation to provide summary and description of each endpoint
  * Use `@ApiResponses` to document all possible response types
  * For each response type, use `@ApiResponse` with:
    * `responseCode` (e.g., "200", "404")
    * `description` of the response
    * `content` specification with mediaType and schema when applicable
  * Use `@Parameter` for method parameters with description and required flag
  * Example:
    ```java
    @Operation(
        summary = "Retrieve a user by ID",
        description = "Retrieves the user resource identified by the given ID"
    )
    @ApiResponses(value = {
        @ApiResponse(
            responseCode = "200",
            description = "User retrieved successfully",
            content = @Content(mediaType = "application/json",
                    schema = @Schema(implementation = UserDto.class))
        ),
        @ApiResponse(responseCode = "400", description = "Invalid ID format", content = @Content),
        @ApiResponse(responseCode = "404", description = "User not found", content = @Content)
    })
    @GetMapping("/{id}")
    ResponseEntity<UserDto> getUser(@Parameter(description = "ID of the user to retrieve", required = true)
                                   @PathVariable UUID id) {
        // Method implementation
    }
    ```

## 12. Internationalization with ResourceBundles
* Externalize all user-facing text such as labels, prompts, and messages into ResourceBundles rather than embedding them in code.

## 13. Use Lombok.

## 14. Prefer package-private over public for Spring components
* Declare Controllers, their request-handling methods, `@Configuration` classes and `@Bean` methods with default (package-private) visibility whenever possible. There's no obligation to make everything `public`.

## 15. Disable Open Session in View Pattern
* While using Spring Data JPA, disable the Open Session in View filter by setting ` spring.jpa.open-in-view=false` in `application.properties/yml.`

## Workflow guideline

## 1. Changes
* Do not change an existing method if no test exists for this code. If a test exists but you do not see any results after executing the test, there might be something wrong with the test configuration within intellij, try to fix it or ask for help.

## 2. Lists
* do not use get(0) but use getFirst() to retrieve the first element only from a list



## more to be defined ##

## 15a. Use Spring Boot DevTools for Development

## 16a. Use Spring Boot's Docker Compose Support

## 17. Use Spring Boot's OAuth2 Client Support

## 18. Use Spring Boot's Prometheus Support

## 19. Use Spring Security for Authentication and Authorization

## 20. Use Spring Session for Distributed Sessions

## 21. Use Spring gRPC for gRPC Services

## 22. Use Spring Web for Web Applications

## 23. Use Spring Data JPA for Database Access

## 24. Use Spring Boot's Configuration Processor

## 25. Use Spring Cloud for Microservices

## 26. Use Spring Cloud Eureka for Service Discovery

## 27. Use Spring Cloud Vault for Configuration Management

## 28. Use Spring Cloud Bootstrap for Application Context Services

## 29. Use Spring Cloud Netflix for Service Discovery and Load Balancing

## 30. Use Spring Cloud OpenFeign for Declarative REST Clients

## 31. Use Spring Cloud Circuit Breaker for Resilience

## 32. Use Spring Cloud Gateway for API Gateway

## 33. Use Spring Cloud Config for Centralized Configuration

## 34. Use Spring Cloud Sleuth for Distributed Tracing
