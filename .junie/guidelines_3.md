# Microservice Application Development Guidelines

## Table of Contents
1. [Project Structure](#project-structure)
2. [Technology Stack Guidelines](#technology-stack-guidelines)
3. [Code Quality Standards](#code-quality-standards)
4. [Testing Strategy](#testing-strategy)
5. [Infrastructure & DevOps](#infrastructure--devops)
6. [Security Guidelines](#security-guidelines)
7. [Performance & Monitoring](#performance--monitoring)
8. [Documentation Standards](#documentation-standards)

## Project Structure

### Root Project Structure
```
microservice-platform/
├── README.md
├── docker-compose.yml
├── docker-compose.dev.yml
├── .gitignore
├── .editorconfig
├── scripts/
│   ├── build-all.sh
│   ├── test-all.sh
│   └── deploy.sh
├── docs/
│   ├── api/
│   ├── architecture/
│   └── deployment/
├── infrastructure/
│   ├── k8s/
│   ├── helm/
│   └── terraform/
├── shared/
│   ├── proto/
│   ├── schemas/
│   └── common-libs/
├── services/
│   ├── auth-service/ (Rust/Actix)
│   ├── trading-service/ (Rust/RustQuant)
│   ├── ml-service/ (Python/JAX/Flax)
│   ├── analytics-service/ (Python/Polars/Norse)
│   ├── gateway-service/ (Java/Spring Boot)
│   └── notification-service/ (Java/Spring Boot)
├── frontends/
│   ├── web-app/ (Angular/TypeScript)
│   ├── mobile-android/ (Java)
│   └── mobile-ios/ (Swift)
└── tests/
    ├── integration/
    ├── e2e/
    └── performance/
```

### Service-Level Structure Standards

#### Rust Services Structure
```
rust-service/
├── Cargo.toml
├── Dockerfile
├── .dockerignore
├── src/
│   ├── main.rs
│   ├── lib.rs
│   ├── config/
│   ├── handlers/
│   ├── models/
│   ├── services/
│   ├── repositories/
│   ├── middleware/
│   └── utils/
├── tests/
│   ├── integration/
│   └── unit/
├── benches/
├── migrations/
└── docs/
```

#### Python Services Structure
```
python-service/
├── pyproject.toml
├── requirements.txt
├── requirements-dev.txt
├── Dockerfile
├── .dockerignore
├── src/
│   └── service_name/
│       ├── __init__.py
│       ├── main.py
│       ├── config/
│       ├── api/
│       ├── core/
│       ├── models/
│       ├── services/
│       └── utils/
├── tests/
│   ├── unit/
│   ├── integration/
│   └── conftest.py
├── scripts/
└── docs/
```

#### Java Services Structure
```
java-service/
├── pom.xml
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
│   │   │       └── exception/
│   │   └── resources/
│   └── test/
├── target/
└── docs/
```

## Technology Stack Guidelines

### Rust Services (Actix Web, RustQuant)

#### Code Standards
- **Error Handling**: Use `Result<T, E>` consistently, create custom error types
- **Async/Await**: Prefer async functions, use `tokio` runtime
- **Serialization**: Use `serde` for JSON/MessagePack serialization
- **Database**: Use `sqlx` with compile-time checked queries
- **Logging**: Use `tracing` with structured logging

```rust
// Example error handling
#[derive(Debug, thiserror::Error)]
pub enum ServiceError {
    #[error("Database error: {0}")]
    Database(#[from] sqlx::Error),
    #[error("Validation error: {message}")]
    Validation { message: String },
}

// Example handler
#[tracing::instrument(skip(pool))]
pub async fn create_user(
    web::Json(user_data): web::Json<CreateUserRequest>,
    pool: web::Data<PgPool>,
) -> Result<HttpResponse, ServiceError> {
    // Implementation
}
```

#### Dependencies Management
- Pin major versions in `Cargo.toml`
- Use `cargo-audit` for security scanning
- Regular dependency updates with `cargo outdated`

### Python Services (JAX, Flax, Polars, Norse)

#### Code Standards
- **Type Hints**: Mandatory for all function signatures
- **Code Formatting**: Use `black` and `isort`
- **Linting**: Use `ruff` for fast linting
- **Dependency Management**: Use `poetry` or `pip-tools`

```python
# Example with proper typing
from typing import Optional, List
import jax.numpy as jnp
from flax import linen as nn

class MLModel(nn.Module):
    features: List[int]
    
    @nn.compact
    def __call__(self, x: jnp.ndarray) -> jnp.ndarray:
        for feat in self.features:
            x = nn.Dense(feat)(x)
            x = nn.relu(x)
        return x

async def predict(
    model: MLModel,
    data: jnp.ndarray,
    batch_size: Optional[int] = None
) -> jnp.ndarray:
    # Implementation
    pass
```

#### Performance Standards
- Use `JAX` JIT compilation for compute-intensive operations
- Implement proper batching for ML workloads
- Use `Polars` for large-scale data processing over Pandas

### Java Services (Spring Boot)

#### Code Standards
- **Java Version**: Use Java 17+ with modern features
- **Code Style**: Follow Google Java Style Guide
- **Dependency Injection**: Use constructor injection
- **Exception Handling**: Create custom exceptions with proper hierarchy

```java
@RestController
@RequestMapping("/api/v1/users")
@Validated
@Slf4j
public class UserController {
    
    private final UserService userService;
    
    public UserController(UserService userService) {
        this.userService = userService;
    }
    
    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    public ResponseEntity<UserDto> createUser(
        @Valid @RequestBody CreateUserRequest request
    ) {
        log.info("Creating user with email: {}", request.getEmail());
        UserDto user = userService.createUser(request);
        return ResponseEntity.created(URI.create("/api/v1/users/" + user.getId()))
                .body(user);
    }
}
```

### Frontend Guidelines

#### Angular/TypeScript
- **Architecture**: Use feature modules with lazy loading
- **State Management**: NgRx for complex state, signals for simple state
- **Testing**: Jest + Testing Library for unit tests, Cypress for E2E
- **Code Quality**: ESLint + Prettier, strict TypeScript config

```typescript
// Example service with proper typing
@Injectable({
  providedIn: 'root'
})
export class UserService {
  private readonly apiUrl = environment.apiUrl;
  
  constructor(private http: HttpClient) {}
  
  createUser(userData: CreateUserRequest): Observable<User> {
    return this.http.post<User>(`${this.apiUrl}/users`, userData)
      .pipe(
        catchError(this.handleError<User>('createUser'))
      );
  }
  
  private handleError<T>(operation = 'operation', result?: T) {
    return (error: any): Observable<T> => {
      console.error(`${operation} failed: ${error.message}`);
      return of(result as T);
    };
  }
}
```

#### Mobile Applications
- **Android**: Follow MVVM pattern, use Kotlin where possible
- **iOS**: Use MVVM or VIPER, SwiftUI for new features
- **Shared Code**: Consider Kotlin Multiplatform for business logic

## Code Quality Standards

### General Principles
1. **SOLID Principles**: Follow all SOLID principles strictly
2. **DRY**: Don't Repeat Yourself - extract common functionality
3. **YAGNI**: You Aren't Gonna Need It - avoid over-engineering
4. **Clean Code**: Self-documenting code with meaningful names

### Naming Conventions
- **Rust**: `snake_case` for functions/variables, `PascalCase` for types
- **Python**: `snake_case` for functions/variables, `PascalCase` for classes
- **Java**: `camelCase` for methods/variables, `PascalCase` for classes
- **TypeScript**: `camelCase` for functions/variables, `PascalCase` for classes/interfaces

### Code Review Standards
- **Size**: Maximum 400 lines per PR
- **Focus**: One feature/fix per PR
- **Tests**: All new code must have tests
- **Documentation**: Update docs for API changes
- **Security**: Security review for authentication/authorization changes

## Testing Strategy

### Testing Pyramid

#### Unit Tests (70%)
- **Coverage**: Minimum 80% code coverage
- **Speed**: All unit tests must run in under 30 seconds
- **Isolation**: Use mocks/stubs for external dependencies

```rust
// Rust unit test example
#[cfg(test)]
mod tests {
    use super::*;
    use mockall::predicate::*;
    
    #[tokio::test]
    async fn test_create_user_success() {
        let mut mock_repo = MockUserRepository::new();
        mock_repo
            .expect_create()
            .with(eq(user_data))
            .times(1)
            .returning(|_| Ok(User::default()));
            
        let service = UserService::new(Box::new(mock_repo));
        let result = service.create_user(user_data).await;
        
        assert!(result.is_ok());
    }
}
```

#### Integration Tests (20%)
- **Database**: Use test containers for database integration
- **APIs**: Test complete request/response cycles
- **Message Queues**: Test async message processing

```python
# Python integration test example
import pytest
from testcontainers.postgres import PostgresContainer

@pytest.fixture(scope="session")
def postgres_container():
    with PostgresContainer("postgres:15") as postgres:
        yield postgres

@pytest.mark.asyncio
async def test_user_repository_integration(postgres_container):
    # Test with real database
    pass
```

#### End-to-End Tests (10%)
- **User Journeys**: Test complete user workflows
- **Cross-Service**: Test service interactions
- **Performance**: Basic performance validation

### Testing Tools by Technology
- **Rust**: `cargo test`, `tokio-test`, `testcontainers-rs`
- **Python**: `pytest`, `pytest-asyncio`, `testcontainers`
- **Java**: `JUnit 5`, `TestContainers`, `WireMock`
- **Angular**: `Jest`, `Cypress`, `Testing Library`

### Security Testing
- **SAST**: Static Application Security Testing in CI/CD
- **DAST**: Dynamic Application Security Testing
- **Dependency Scanning**: Regular vulnerability scans
- **Penetration Testing**: Quarterly security assessments

### Performance Testing
- **Load Testing**: Use `k6` or `Artillery` for API load testing
- **Stress Testing**: Determine breaking points
- **Benchmarking**: Regular performance regression testing

```javascript
// k6 load test example
import http from 'k6/http';
import { check } from 'k6';

export let options = {
  stages: [
    { duration: '2m', target: 100 },
    { duration: '5m', target: 100 },
    { duration: '2m', target: 200 },
    { duration: '5m', target: 200 },
    { duration: '2m', target: 0 },
  ],
};

export default function() {
  let response = http.get('http://api.example.com/users');
  check(response, {
    'status is 200': (r) => r.status === 200,
    'response time < 500ms': (r) => r.timings.duration < 500,
  });
}
```

## Infrastructure & DevOps

### Containerization
- **Docker**: Multi-stage builds for smaller images
- **Base Images**: Use distroless or Alpine images
- **Security**: Regular base image updates, vulnerability scanning

```dockerfile
# Multi-stage Rust Dockerfile
FROM rust:1.70 as builder
WORKDIR /app
COPY Cargo.toml Cargo.lock ./
RUN cargo fetch
COPY src ./src
RUN cargo build --release

FROM gcr.io/distroless/cc-debian11
COPY --from=builder /app/target/release/service /service
EXPOSE 8080
USER 1000
ENTRYPOINT ["/service"]
```

### Kubernetes Deployment
- **Helm Charts**: Use Helm for application deployment
- **Resource Limits**: Set CPU/memory limits for all containers
- **Health Checks**: Implement readiness/liveness probes
- **Security**: Use Pod Security Standards

```yaml
# Example Kubernetes deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: user-service
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
      securityContext:
        runAsNonRoot: true
        runAsUser: 1000
      containers:
      - name: user-service
        image: user-service:latest
        ports:
        - containerPort: 8080
        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"
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
```

### CI/CD Pipeline
- **Build**: Parallel builds for different technologies
- **Test**: Run all test suites with proper reporting
- **Security**: SAST, DAST, and dependency scanning
- **Deploy**: Blue-green or canary deployments

### Monitoring & Observability
- **Metrics**: Prometheus + Grafana
- **Logging**: ELK Stack or Loki
- **Tracing**: Jaeger or Zipkin
- **Alerting**: AlertManager with PagerDuty integration

## Security Guidelines

### Authentication & Authorization
- **JWT**: Use short-lived access tokens with refresh tokens
- **RBAC**: Role-based access control for all services
- **API Keys**: For service-to-service communication
- **OAuth 2.0**: For third-party integrations

### Data Protection
- **Encryption**: TLS 1.3 for all communications
- **At Rest**: Encrypt sensitive data in databases
- **PII**: Proper handling of personally identifiable information
- **Audit Logs**: Log all security-relevant events

### Security Headers
```http
Strict-Transport-Security: max-age=31536000; includeSubDomains
Content-Security-Policy: default-src 'self'
X-Frame-Options: DENY
X-Content-Type-Options: nosniff
Referrer-Policy: strict-origin-when-cross-origin
```

### Input Validation
- **Sanitization**: Sanitize all user inputs
- **Validation**: Server-side validation for all requests
- **Rate Limiting**: Implement rate limiting on all APIs
- **OWASP**: Follow OWASP Top 10 guidelines

## Performance & Monitoring

### Performance Targets
- **API Response Time**: 95th percentile < 200ms
- **Database Queries**: < 100ms average
- **Memory Usage**: < 80% of allocated resources
- **CPU Usage**: < 70% average load

### Monitoring Metrics
- **Golden Signals**: Latency, Traffic, Errors, Saturation
- **Business Metrics**: User actions, revenue-related metrics
- **Infrastructure Metrics**: CPU, memory, disk, network

### Caching Strategy
- **Redis**: For session data and frequently accessed data
- **CDN**: For static assets and API responses
- **Application-Level**: In-memory caching for expensive operations

## Documentation Standards

### API Documentation
- **OpenAPI 3.0**: All REST APIs must have OpenAPI specs
- **Examples**: Provide request/response examples
- **Error Codes**: Document all possible error responses
- **Versioning**: Clear API versioning strategy

### Code Documentation
- **README**: Each service must have comprehensive README
- **Code Comments**: Document complex business logic
- **Architecture Decisions**: Use ADRs (Architecture Decision Records)
- **Runbooks**: Operational procedures and troubleshooting guides

### Changelog
- **Semantic Versioning**: Follow semver for all releases
- **Keep a Changelog**: Maintain CHANGELOG.md files
- **Release Notes**: Detailed release notes for stakeholders

## IntelliJ IDEA Project Setup

### Module Configuration
1. Create separate modules for each service/frontend
2. Configure appropriate SDKs for each technology
3. Set up shared code style configurations
4. Configure run configurations for each service

### Recommended Plugins
- **Rust**: Rust Plugin
- **Python**: Python Plugin
- **Angular**: Angular and TypeScript Plugin
- **Docker**: Docker Plugin
- **Kubernetes**: Kubernetes Plugin
- **Database**: Database Tools and SQL Plugin

### Code Style Configuration
- Import shared code style configurations
- Enable automatic formatting on save
- Configure inspection profiles for each language
- Set up file templates for consistency

## Development Workflow

### Git Workflow
- **Branching**: GitFlow with feature branches
- **Commits**: Conventional commits format
- **Reviews**: Mandatory code reviews for all changes
- **CI/CD**: Automated testing and deployment

### Local Development
- **Docker Compose**: For local service dependencies
- **Hot Reload**: Enable for faster development cycles
- **Environment Variables**: Use `.env` files for configuration
- **Database Migrations**: Automated migration scripts

### Release Process
1. **Feature Freeze**: Code freeze before release
2. **Testing**: Comprehensive testing across all environments
3. **Deployment**: Staged rollouts with monitoring
4. **Rollback**: Quick rollback procedures if issues arise

---

## Enforcement

These guidelines are **MANDATORY** for all team members. Any deviation must be approved by the technical lead and documented with reasoning. Regular reviews will be conducted to ensure compliance and update guidelines as needed.

**Remember**: These guidelines exist to ensure code quality, maintainability, security, and team productivity. Following them strictly will result in a robust, scalable, and maintainable microservice platform.