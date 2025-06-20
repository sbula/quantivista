# Microservices Development Guidelines

## Table of Contents

1. [Project Structure](#project-structure)
2. [General Best Practices](#general-best-practices)
3. [Rust Microservices](#rust-microservices)
4. [Python Microservices](#python-microservices)
5. [Java Microservices](#java-microservices)
6. [Infrastructure & DevOps](#infrastructure--devops)
7. [Testing Strategy](#testing-strategy)
8. [Communication Patterns](#communication-patterns)
9. [Monitoring & Observability](#monitoring--observability)
10. [Security Guidelines](#security-guidelines)

## Project Structure

### Root Directory Layout

```
microservices-platform/
├── .gitignore
├── .editorconfig
├── docker-compose.yml
├── docker-compose.override.yml
├── README.md
├── Makefile
├── scripts/
│   ├── build-all.sh
│   ├── test-all.sh
│   └── deploy.sh
├── infrastructure/
│   ├── kubernetes/
│   ├── terraform/
│   └── docker/
├── shared/
│   ├── proto/                    # Protocol Buffers definitions
│   ├── schemas/                  # OpenAPI/AsyncAPI specs
│   └── configs/                  # Shared configuration templates
├── services/
│   ├── rust/
│   │   ├── user-service/
│   │   ├── auth-service/
│   │   └── payment-service/
│   ├── python/
│   │   ├── notification-service/
│   │   ├── analytics-service/
│   │   └── ml-service/
│   └── java/
│       ├── order-service/
│       ├── inventory-service/
│       └── gateway-service/
├── tools/
│   ├── code-gen/
│   └── migration-scripts/
└── docs/
    ├── api/
    ├── architecture/
    └── deployment/
```

### IntelliJ Project Configuration

- **Root Project**: Set as multi-module project
- **Module Structure**: Each service as separate module
- **Build Tools**:
  - Rust: Cargo workspaces
  - Python: Poetry/pip-tools
  - Java: Gradle multi-project
- **IDE Plugins Required**:
  - Rust Plugin
  - Python Plugin
  - Docker Plugin
  - Kubernetes Plugin

## General Best Practices

### Code Quality Standards

1. **Consistent Formatting**

   - Use language-specific formatters (rustfmt, black, google-java-format)
   - Configure pre-commit hooks for all repositories
   - Maintain .editorconfig for consistent indentation

2. **Documentation**

   - Every service MUST have README.md with:
     - Purpose and responsibilities
     - API documentation
     - Setup instructions
     - Dependencies
   - Code documentation for public APIs
   - Architecture Decision Records (ADRs) for significant changes

3. **Version Control**
   - Conventional commit messages
   - Feature branch workflow
   - Mandatory code reviews
   - Squash merges to main branch

### API Design

1. **RESTful Design**

   - Use HTTP status codes correctly
   - Implement proper HATEOAS where applicable
   - Version APIs using URL versioning (/v1/, /v2/)
   - Use OpenAPI 3.0 specifications

2. **GraphQL (if applicable)**

   - Schema-first approach
   - Implement proper error handling
   - Use DataLoader pattern for N+1 queries

3. **Event-Driven Architecture**
   - Use AsyncAPI for event documentation
   - Implement idempotent event handlers
   - Use correlation IDs for tracing

## Rust Microservices

### Project Structure

```
rust-service/
├── Cargo.toml
├── Cargo.lock
├── README.md
├── Dockerfile
├── .dockerignore
├── src/
│   ├── main.rs
│   ├── lib.rs
│   ├── config/
│   │   └── mod.rs
│   ├── handlers/
│   │   ├── mod.rs
│   │   └── user.rs
│   ├── models/
│   │   ├── mod.rs
│   │   └── user.rs
│   ├── services/
│   │   ├── mod.rs
│   │   └── user_service.rs
│   ├── repositories/
│   │   ├── mod.rs
│   │   └── user_repository.rs
│   ├── middleware/
│   │   └── mod.rs
│   └── utils/
│       └── mod.rs
├── tests/
│   ├── integration/
│   └── unit/
├── benches/
└── migrations/
```

### Coding Standards

1. **Dependencies**

   ```toml
   [dependencies]
   tokio = { version = "1.0", features = ["full"] }
   serde = { version = "1.0", features = ["derive"] }
   serde_json = "1.0"
   reqwest = { version = "0.11", features = ["json"] }
   sqlx = { version = "0.7", features = ["runtime-tokio-rustls", "postgres", "chrono", "uuid"] }
   tracing = "0.1"
   tracing-subscriber = "0.3"
   uuid = { version = "1.0", features = ["v4", "serde"] }
   ```

2. **Error Handling**

   - Use `thiserror` for custom error types
   - Implement proper error propagation with `?` operator
   - Never use `unwrap()` or `expect()` in production code
   - Use `anyhow` for application-level errors

3. **Async Programming**

   - Use `tokio` as the async runtime
   - Implement proper connection pooling
   - Use `Arc<T>` for shared state
   - Avoid blocking operations in async contexts

4. **Performance**
   - Use `cargo flamegraph` for profiling
   - Implement proper caching strategies
   - Use connection pooling for databases
   - Consider using `deadpool` for resource pooling

### Configuration Management

```rust
use serde::{Deserialize, Serialize};
use std::env;

#[derive(Debug, Deserialize, Serialize, Clone)]
pub struct Config {
    pub server: ServerConfig,
    pub database: DatabaseConfig,
    pub redis: RedisConfig,
}

#[derive(Debug, Deserialize, Serialize, Clone)]
pub struct ServerConfig {
    pub host: String,
    pub port: u16,
}

impl Config {
    pub fn from_env() -> Result<Self, config::ConfigError> {
        let mut cfg = config::Config::builder()
            .add_source(config::Environment::with_prefix("APP"))
            .build()?;

        cfg.try_deserialize()
    }
}
```

## Python Microservices

### Project Structure

```
python-service/
├── pyproject.toml
├── poetry.lock
├── README.md
├── Dockerfile
├── .dockerignore
├── .env.example
├── app/
│   ├── __init__.py
│   ├── main.py
│   ├── config/
│   │   ├── __init__.py
│   │   └── settings.py
│   ├── api/
│   │   ├── __init__.py
│   │   ├── dependencies.py
│   │   └── v1/
│   │       ├── __init__.py
│   │       ├── endpoints/
│   │       └── models/
│   ├── core/
│   │   ├── __init__.py
│   │   ├── security.py
│   │   ├── database.py
│   │   └── exceptions.py
│   ├── services/
│   │   ├── __init__.py
│   │   └── user_service.py
│   ├── repositories/
│   │   ├── __init__.py
│   │   └── user_repository.py
│   └── utils/
│       ├── __init__.py
│       └── helpers.py
├── tests/
│   ├── __init__.py
│   ├── conftest.py
│   ├── unit/
│   └── integration/
├── migrations/
└── scripts/
```

### Coding Standards

1. **Dependencies Management**

   ```toml
   [tool.poetry.dependencies]
   python = "^3.11"
   fastapi = "^0.104.0"
   uvicorn = {extras = ["standard"], version = "^0.24.0"}
   pydantic = {extras = ["email"], version = "^2.4.0"}
   sqlalchemy = "^2.0.0"
   alembic = "^1.12.0"
   redis = "^5.0.0"
   celery = "^5.3.0"
   httpx = "^0.25.0"
   structlog = "^23.1.0"
   ```

2. **Code Quality Tools**

   ```toml
   [tool.poetry.group.dev.dependencies]
   pytest = "^7.4.0"
   pytest-asyncio = "^0.21.0"
   black = "^23.9.0"
   isort = "^5.12.0"
   flake8 = "^6.1.0"
   mypy = "^1.6.0"
   pre-commit = "^3.5.0"
   ```

3. **Type Hints**

   - Use type hints for all function signatures
   - Use `from __future__ import annotations` for forward references
   - Configure mypy strict mode
   - Use Pydantic models for data validation

4. **Error Handling**

   ```python
   from fastapi import HTTPException
   from typing import Optional
   import structlog

   logger = structlog.get_logger()

   class ServiceError(Exception):
       def __init__(self, message: str, status_code: int = 500):
           self.message = message
           self.status_code = status_code
           super().__init__(self.message)

   async def handle_service_error(error: Exception) -> HTTPException:
       logger.error("Service error occurred", error=str(error))
       if isinstance(error, ServiceError):
           raise HTTPException(status_code=error.status_code, detail=error.message)
       raise HTTPException(status_code=500, detail="Internal server error")
   ```

### FastAPI Best Practices

1. **Application Structure**

   ```python
   from fastapi import FastAPI, Depends
   from fastapi.middleware.cors import CORSMiddleware
   from fastapi.middleware.trustedhost import TrustedHostMiddleware
   import structlog

   def create_app() -> FastAPI:
       app = FastAPI(
           title="Service Name",
           description="Service description",
           version="1.0.0",
           docs_url="/docs",
           redoc_url="/redoc"
       )

       # Add middleware
       app.add_middleware(
           CORSMiddleware,
           allow_origins=["*"],  # Configure properly for production
           allow_credentials=True,
           allow_methods=["*"],
           allow_headers=["*"],
       )

       # Include routers
       from app.api.v1 import router as api_router
       app.include_router(api_router, prefix="/api/v1")

       return app
   ```

## Java Microservices

### Project Structure

```
java-service/
├── build.gradle
├── gradle.properties
├── settings.gradle
├── README.md
├── Dockerfile
├── .dockerignore
├── src/
│   ├── main/
│   │   ├── java/
│   │   │   └── com/company/service/
│   │   │       ├── Application.java
│   │   │       ├── config/
│   │   │       ├── controller/
│   │   │       ├── service/
│   │   │       ├── repository/
│   │   │       ├── model/
│   │   │       ├── dto/
│   │   │       ├── exception/
│   │   │       └── util/
│   │   └── resources/
│   │       ├── application.yml
│   │       ├── application-dev.yml
│   │       ├── application-prod.yml
│   │       └── db/migration/
│   └── test/
│       ├── java/
│       └── resources/
└── gradle/
```

### Coding Standards

1. **Dependencies (build.gradle)**

   ```gradle
   dependencies {
       implementation 'org.springframework.boot:spring-boot-starter-web'
       implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
       implementation 'org.springframework.boot:spring-boot-starter-security'
       implementation 'org.springframework.boot:spring-boot-starter-actuator'
       implementation 'org.springframework.boot:spring-boot-starter-validation'
       implementation 'org.springframework.cloud:spring-cloud-starter-netflix-eureka-client'
       implementation 'org.springframework.cloud:spring-cloud-starter-config'
       implementation 'io.micrometer:micrometer-registry-prometheus'
       implementation 'org.postgresql:postgresql'
       implementation 'org.flywaydb:flyway-core'
       implementation 'org.mapstruct:mapstruct:1.5.5.Final'

       testImplementation 'org.springframework.boot:spring-boot-starter-test'
       testImplementation 'org.testcontainers:junit-jupiter'
       testImplementation 'org.testcontainers:postgresql'
   }
   ```

2. **Configuration Management**

   ```java
   @ConfigurationProperties(prefix = "app")
   @Data
   @Validated
   public class ApplicationProperties {

       @NotNull
       private String name;

       @Valid
       private Database database = new Database();

       @Valid
       private Security security = new Security();

       @Data
       public static class Database {
           private int maxPoolSize = 10;
           private Duration connectionTimeout = Duration.ofSeconds(30);
       }

       @Data
       public static class Security {
           @NotEmpty
           private String jwtSecret;
           private Duration jwtExpiration = Duration.ofHours(24);
       }
   }
   ```

3. **Error Handling**

   ```java
   @RestControllerAdvice
   @Slf4j
   public class GlobalExceptionHandler {

       @ExceptionHandler(ValidationException.class)
       public ResponseEntity<ErrorResponse> handleValidation(ValidationException ex) {
           log.warn("Validation error: {}", ex.getMessage());
           return ResponseEntity.badRequest()
               .body(new ErrorResponse("VALIDATION_ERROR", ex.getMessage()));
       }

       @ExceptionHandler(ResourceNotFoundException.class)
       public ResponseEntity<ErrorResponse> handleNotFound(ResourceNotFoundException ex) {
           log.warn("Resource not found: {}", ex.getMessage());
           return ResponseEntity.notFound().build();
       }

       @ExceptionHandler(Exception.class)
       public ResponseEntity<ErrorResponse> handleGeneral(Exception ex) {
           log.error("Unexpected error", ex);
           return ResponseEntity.internalServerError()
               .body(new ErrorResponse("INTERNAL_ERROR", "An unexpected error occurred"));
       }
   }
   ```

4. **Service Layer Pattern**
   ```java
   @Service
   @Transactional
   @RequiredArgsConstructor
   @Slf4j
   public class UserService {

       private final UserRepository userRepository;
       private final UserMapper userMapper;
       private final ApplicationEventPublisher eventPublisher;

       @Transactional(readOnly = true)
       public UserDto findById(Long id) {
           return userRepository.findById(id)
               .map(userMapper::toDto)
               .orElseThrow(() -> new ResourceNotFoundException("User not found"));
       }

       public UserDto create(CreateUserRequest request) {
           User user = userMapper.toEntity(request);
           User savedUser = userRepository.save(user);

           eventPublisher.publishEvent(new UserCreatedEvent(savedUser.getId()));

           return userMapper.toDto(savedUser);
       }
   }
   ```

## Infrastructure & DevOps

### Docker Configuration

1. **Multi-stage Dockerfile (Rust)**

   ```dockerfile
   FROM rust:1.75 as builder
   WORKDIR /app
   COPY Cargo.toml Cargo.lock ./
   RUN mkdir src && echo "fn main() {}" > src/main.rs
   RUN cargo build --release
   COPY src ./src
   RUN cargo build --release

   FROM debian:bookworm-slim
   RUN apt-get update && apt-get install -y ca-certificates && rm -rf /var/lib/apt/lists/*
   COPY --from=builder /app/target/release/service /usr/local/bin/service
   EXPOSE 8080
   CMD ["service"]
   ```

2. **Multi-stage Dockerfile (Python)**

   ```dockerfile
   FROM python:3.11-slim as builder
   WORKDIR /app
   COPY pyproject.toml poetry.lock ./
   RUN pip install poetry && poetry config virtualenvs.create false
   RUN poetry install --only=main --no-dev

   FROM python:3.11-slim
   WORKDIR /app
   COPY --from=builder /usr/local/lib/python3.11/site-packages /usr/local/lib/python3.11/site-packages
   COPY app ./app
   EXPOSE 8000
   CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
   ```

3. **Multi-stage Dockerfile (Java)**

   ```dockerfile
   FROM openjdk:17-jdk-slim as builder
   WORKDIR /app
   COPY gradle/ gradle/
   COPY build.gradle settings.gradle gradlew ./
   RUN ./gradlew build -x test --no-daemon
   COPY src ./src
   RUN ./gradlew build --no-daemon

   FROM openjdk:17-jre-slim
   WORKDIR /app
   COPY --from=builder /app/build/libs/*.jar app.jar
   EXPOSE 8080
   CMD ["java", "-jar", "app.jar"]
   ```

### Kubernetes Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: user-service
  labels:
    app: user-service
spec:
  replicas: 3
  selector:
    matchLabels:
      app: user-service
  template:
    metadata:
      labels:
        app: user-service
    spec:
      containers:
        - name: user-service
          image: user-service:latest
          ports:
            - containerPort: 8080
          env:
            - name: DATABASE_URL
              valueFrom:
                secretKeyRef:
                  name: db-secret
                  key: url
          livenessProbe:
            httpGet:
              path: /health
              port: 8080
            initialDelaySeconds: 30
            periodSeconds: 10
          readinessProbe:
            httpGet:
              path: /ready
              port: 8080
            initialDelaySeconds: 5
            periodSeconds: 5
          resources:
            requests:
              memory: "256Mi"
              cpu: "250m"
            limits:
              memory: "512Mi"
              cpu: "500m"
```

### CI/CD Pipeline (.github/workflows/service.yml)

```yaml
name: Service CI/CD
on:
  push:
    branches: [main, develop]
    paths: ["services/rust/user-service/**"]
  pull_request:
    branches: [main]
    paths: ["services/rust/user-service/**"]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Setup Rust
        uses: actions-rs/toolchain@v1
        with:
          toolchain: stable
      - name: Run tests
        run: cd services/rust/user-service && cargo test
      - name: Run clippy
        run: cd services/rust/user-service && cargo clippy -- -D warnings

  build:
    needs: test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    steps:
      - uses: actions/checkout@v4
      - name: Build and push Docker image
        uses: docker/build-push-action@v5
        with:
          context: services/rust/user-service
          push: true
          tags: user-service:${{ github.sha }}
```

## Testing Strategy

### Unit Testing

1. **Rust Testing**

   ```rust
   #[cfg(test)]
   mod tests {
       use super::*;
       use tokio_test;

       #[tokio::test]
       async fn test_user_creation() {
           let user_service = UserService::new();
           let request = CreateUserRequest {
               email: "test@example.com".to_string(),
               name: "Test User".to_string(),
           };

           let result = user_service.create_user(request).await;
           assert!(result.is_ok());
       }
   }
   ```

2. **Python Testing**

   ```python
   import pytest
   from httpx import AsyncClient
   from app.main import app

   @pytest.mark.asyncio
   async def test_create_user():
       async with AsyncClient(app=app, base_url="http://test") as ac:
           response = await ac.post("/users", json={
               "email": "test@example.com",
               "name": "Test User"
           })
       assert response.status_code == 201
   ```

3. **Java Testing**
   ```java
   @SpringBootTest
   @AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE)
   @Testcontainers
   class UserServiceTest {

       @Container
       static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:15")
               .withDatabaseName("testdb")
               .withUsername("test")
               .withPassword("test");

       @Autowired
       private UserService userService;

       @Test
       void shouldCreateUser() {
           CreateUserRequest request = new CreateUserRequest("test@example.com", "Test User");
           UserDto result = userService.create(request);

           assertThat(result.getEmail()).isEqualTo("test@example.com");
       }
   }
   ```

### Integration Testing

- Use TestContainers for database testing
- Implement contract testing with Pact
- Use WireMock for external service mocking
- Implement end-to-end testing with real databases

### Performance Testing

- Use Apache JMeter or k6 for load testing
- Implement benchmark tests in each language
- Monitor response times and throughput
- Set up performance regression testing

## Communication Patterns

### Synchronous Communication

1. **HTTP/REST**

   - Use OpenAPI specifications
   - Implement proper HTTP status codes
   - Use correlation IDs for tracing
   - Implement circuit breakers

2. **gRPC**
   - Define proto files in shared directory
   - Use proper error handling
   - Implement streaming where appropriate
   - Use interceptors for cross-cutting concerns

### Asynchronous Communication

1. **Message Queues (RabbitMQ/Apache Kafka)**

   - Use dead letter queues
   - Implement idempotent consumers
   - Use message schemas (Avro/JSON Schema)
   - Implement proper retry mechanisms

2. **Event Sourcing**
   - Design events as immutable facts
   - Use event versioning strategies
   - Implement event replay capabilities
   - Separate read and write models (CQRS)

## Monitoring & Observability

### Logging

1. **Structured Logging**

   - Use JSON format for logs
   - Include correlation IDs
   - Log at appropriate levels
   - Never log sensitive information

2. **Centralized Logging**
   - Use ELK Stack or similar
   - Implement log aggregation
   - Set up log retention policies
   - Create log-based alerts

### Metrics

1. **Application Metrics**

   - Use Prometheus format
   - Implement RED metrics (Rate, Errors, Duration)
   - Monitor business metrics
   - Set up custom dashboards

2. **Infrastructure Metrics**
   - Monitor CPU, memory, disk usage
   - Track network metrics
   - Monitor database performance
   - Set up capacity planning alerts

### Tracing

1. **Distributed Tracing**
   - Use OpenTelemetry standard
   - Implement trace sampling
   - Track cross-service calls
   - Use trace-based debugging

### Health Checks

```yaml
# Health check endpoints
GET /health       # Basic health status
GET /health/live  # Liveness probe
GET /health/ready # Readiness probe
GET /metrics      # Prometheus metrics
```

## Security Guidelines

### Authentication & Authorization

1. **JWT Tokens**

   - Use RS256 algorithm
   - Implement token refresh
   - Set appropriate expiration times
   - Validate all claims

2. **API Security**
   - Implement rate limiting
   - Use HTTPS everywhere
   - Validate all inputs
   - Implement CORS properly

### Data Protection

1. **Encryption**

   - Encrypt data at rest
   - Use TLS for data in transit
   - Implement field-level encryption for PII
   - Use secure key management

2. **Input Validation**
   - Validate all inputs at API boundaries
   - Use parameterized queries
   - Implement CSRF protection
   - Sanitize outputs

### Secrets Management

- Use Kubernetes secrets or HashiCorp Vault
- Never commit secrets to version control
- Rotate secrets regularly
- Use least privilege principle

### Security Scanning

- Implement dependency vulnerability scanning
- Use static code analysis tools
- Perform regular security audits
- Implement security testing in CI/CD

---

## Enforcement Rules

### Code Review Requirements

- All code must be reviewed by at least 2 team members
- Security-related changes require security team approval
- Performance-critical changes require performance testing
- Breaking changes require architecture team approval

### Definition of Done

- [ ] Code passes all tests (unit, integration, security)
- [ ] Code is properly documented
- [ ] API documentation is updated
- [ ] Performance benchmarks pass
- [ ] Security scan passes
- [ ] Code review approved
- [ ] Changes are backward compatible or properly versioned

### Continuous Improvement

- Weekly tech debt review meetings
- Monthly architecture review sessions
- Quarterly security assessments
- Regular performance optimization reviews

This document is a living guideline and should be updated as the project evolves and new best practices emerge.
