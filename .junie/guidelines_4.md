# Microservice Development Guidelines

## Table of Contents

1. [Project Structure & Setup](#project-structure--setup)
2. [Architecture Principles](#architecture-principles)
3. [Technology Stack Guidelines](#technology-stack-guidelines)
4. [API Design & Communication](#api-design--communication)
5. [Event Sourcing & CQRS Implementation](#event-sourcing--cqrs-implementation)
6. [Testing Strategy](#testing-strategy)
7. [Infrastructure as Code](#infrastructure-as-code)
8. [Security & IAM](#security--iam)
9. [Monitoring & Observability](#monitoring--observability)
10. [Development Workflow](#development-workflow)

## Project Structure & Setup

### IntelliJ Multi-Module Project Structure

```
microservices-platform/
├── pom.xml (parent)
├── docker-compose.yml
├── k8s/
├── infrastructure/
│   ├── terraform/
│   └── helm/
├── services/
│   ├── rust-services/
│   │   ├── trading-engine/ (Actix Web + RustQuant)
│   │   ├── risk-calculator/
│   │   └── shared-libs/
│   ├── python-services/
│   │   ├── ml-inference/ (JAX + Flax)
│   │   ├── data-processor/ (Polars + Norse)
│   │   └── requirements/
│   ├── java-services/
│   │   ├── user-service/ (Spring Boot)
│   │   ├── order-service/
│   │   └── shared/
│   └── shared/
│       ├── proto/
│       ├── events/
│       └── schemas/
├── frontends/
│   ├── web-app/ (Angular + TypeScript)
│   ├── android-app/ (Java)
│   └── ios-app/ (Swift)
├── api-gateway/
├── event-store/
└── docs/
    ├── api/
    └── architecture/
```

### Module Configuration

- **Parent Module**: Maven/Gradle multi-module setup with shared dependencies
- **Service Modules**: Each microservice as independent deployable module
- **Shared Libraries**: Common utilities, proto definitions, event schemas
- **Frontend Modules**: Separate modules for each client application

## Architecture Principles

### Core Principles

1. **API-First Design**: Use API-based design to ensure every microservice has APIs so that it can easily communicate with other microservices
2. **Domain-Driven Design**: Services aligned with business domains
3. **Bounded Contexts**: Clear service boundaries with minimal coupling
4. **Event-Driven Architecture**: Asynchronous communication via events
5. **CQRS Pattern**: Define a queryable replica that is kept up to date by subscribing to events published by the services that own the data
6. **No Direct Database Sharing**: Each service owns its data

### Service Design Guidelines

- **Single Responsibility**: One business capability per service
- **Autonomous**: Independent deployment and scaling
- **Stateless**: Store state in databases, not in memory
- **Idempotent**: Operations can be safely retried
- **Fault Tolerant**: Handle failures gracefully

## Technology Stack Guidelines

### Rust Services (Actix Web + RustQuant)

```rust
// Cargo.toml structure
[dependencies]
actix-web = "4.0"
rustquant = "0.10"
tokio = { version = "1.0", features = ["full"] }
serde = { version = "1.0", features = ["derive"] }
tracing = "0.1"
```

**Best Practices:**

- Use `actix-web` for high-performance HTTP services
- Implement async/await patterns consistently
- Use `RustQuant` for financial calculations
- Leverage Rust's ownership system for memory safety
- Use `tracing` for structured logging

### Python Services (JAX + Flax + Polars + Norse)

```python
# requirements.txt structure
jax>=0.4.0
flax>=0.8.0
polars>=0.20.0
norse>=1.0.0
fastapi>=0.100.0
pydantic>=2.0.0
```

**Best Practices:**

- Use `FastAPI` for API endpoints with automatic OpenAPI generation
- Implement JAX for numerical computations
- Use Flax for neural network models
- Leverage Polars for high-performance data processing
- Use Norse for spiking neural networks
- Type hints for all functions
- Async/await for I/O operations

### Java Services (Spring Boot)

```xml
<!-- Key dependencies -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-webflux</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-r2dbc</artifactId>
</dependency>
```

**Best Practices:**

- Use Spring WebFlux for reactive programming
- Implement R2DBC for non-blocking database access
- Use Spring Cloud Gateway for API gateway
- Implement Circuit Breaker pattern with Resilience4j
- Use Spring Security for authentication/authorization

## API Design & Communication

### External Communication (REST via API Gateway)

- **OpenAPI 3.0 Specification**: You can use OpenAPI to create a style guide to suggest how endpoints, requests, responses, and fields should be implemented
- **RESTful Principles**: Resource-based URLs, HTTP verbs
- **Versioning**: URL path versioning (`/api/v1/`)
- **Error Handling**: Consistent error response format
- **Rate Limiting**: Implement throttling at gateway level

### Internal Communication (gRPC)

The most practical approach is often a hybrid strategy: use gRPC for internal service-to-service communication while maintaining REST endpoints for browser clients and public API consumers

**Proto Definition Structure:**

```protobuf
syntax = "proto3";

package trading.v1;

service TradingService {
  rpc CreateOrder(CreateOrderRequest) returns (CreateOrderResponse);
  rpc GetOrder(GetOrderRequest) returns (GetOrderResponse);
}

message CreateOrderRequest {
  string user_id = 1;
  string symbol = 2;
  int64 quantity = 3;
  double price = 4;
}
```

**gRPC Best Practices:**

- Use protocol buffers for message definition
- Implement health checks for all gRPC services
- Use interceptors for cross-cutting concerns
- Implement proper error handling with gRPC status codes
- Use streaming for real-time data when needed

### API Documentation

- **OpenAPI/Swagger**: Auto-generate from code annotations
- **AsyncAPI**: For event-driven APIs
- **gRPC Documentation**: Generate from proto files
- **Postman Collections**: For testing and examples

## Event Sourcing & CQRS Implementation

### Event Sourcing Pattern

Event sourcing persists the state of a business entity such an Order or a Customer as a sequence of state-changing events. Whenever the state of a business entity changes, a new event is appended to the list of events

**Event Store Structure:**

```json
{
  "eventId": "uuid",
  "streamId": "order-12345",
  "eventType": "OrderCreated",
  "eventData": {
    /* event payload */
  },
  "eventVersion": 1,
  "timestamp": "2025-06-20T10:00:00Z",
  "metadata": {
    /* correlation info */
  }
}
```

### CQRS Implementation

In event-sourcing, any event triggered will be stored in an event store. There is no update or delete operations on the data, and every event generated will be stored as a record in the database

**Command Side:**

- Handle commands and generate events
- No direct queries on command side
- Validate business rules before event creation

**Query Side:**

- Subscribe to events and build read models
- Optimized for specific query patterns
- Eventually consistent with command side

### Event Design Principles

- **Past Tense Naming**: `OrderCreated`, `PaymentProcessed`
- **Immutable Events**: Never modify published events
- **Schema Evolution**: Design for backward compatibility
- **Event Correlation**: Track causation and correlation IDs

## Testing Strategy

### Testing Pyramid

#### Unit Testing (70%)

**Rust:**

```rust
#[cfg(test)]
mod tests {
    use super::*;

    #[tokio::test]
    async fn test_calculate_risk() {
        // Test individual functions
    }
}
```

**Python:**

```python
import pytest
from unittest.mock import AsyncMock

@pytest.mark.asyncio
async def test_ml_inference():
    # Test ML model inference
    pass
```

**Java:**

```java
@ExtendWith(MockitoExtension.class)
class OrderServiceTest {
    @Test
    void shouldCreateOrder() {
        // Test business logic
    }
}
```

#### Integration Testing (20%)

Test strategies like unit testing, integration testing, and end-to-end testing

- **Database Integration**: Test with real databases using Testcontainers
- **Message Integration**: Test event publishing/consuming
- **gRPC Integration**: Mocking gRPC services allows you to validate gRPC integration code during your tests

#### End-to-End Testing (10%)

- **API Gateway Testing**: Full request/response cycle
- **Cross-Service Integration**: Multi-service workflows
- **Performance Testing**: Load and stress testing

### Testing Tools by Technology

**Microservices Testing Tools (2025):**
Microservices testing involves testing each microservice separately and with other services

- **Contract Testing**: Pact for consumer-driven contracts
- **Service Virtualization**: WireMock for external dependencies
- **Load Testing**: K6, JMeter for performance testing
- **Chaos Engineering**: Chaos Monkey for resilience testing

### Test Data Management

- **Test Fixtures**: Consistent test data across services
- **Data Anonymization**: Protect sensitive data in tests
- **Environment Isolation**: Separate test environments per service

## Infrastructure as Code

### Terraform Structure

```hcl
# main.tf
provider "aws" {
  region = var.aws_region
}

module "eks" {
  source = "./modules/eks"
  cluster_name = var.cluster_name
}

module "rds" {
  source = "./modules/rds"
  # Event store database
}
```

### Kubernetes Manifests

**Service Deployment:**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: trading-service
spec:
  replicas: 3
  selector:
    matchLabels:
      app: trading-service
  template:
    spec:
      containers:
        - name: trading-service
          image: trading-service:latest
          ports:
            - containerPort: 8080
```

### Helm Charts

- **Service Templates**: Reusable deployment templates
- **Configuration Management**: Environment-specific values
- **Dependencies**: Manage service dependencies

### Infrastructure Testing

To take full advantage of IaC, perform infrastructure-as-code testing prior to production deployments

- **Static Analysis**: Terraform plan validation
- **Unit Testing**: Terratest for infrastructure testing
- **Integration Testing**: Actual resource provisioning tests

## Security & IAM

### Open Source Cloud-Native IAM Solutions

**Recommended Stack:**

1. **Keycloak**: Identity and access management
2. **Open Policy Agent (OPA)**: Policy-based authorization
3. **Istio Service Mesh**: mTLS and traffic security
4. **Falco**: Runtime security monitoring

### Security Implementation

**JWT Token Structure:**

```json
{
  "sub": "user123",
  "iss": "keycloak",
  "aud": ["trading-service", "user-service"],
  "exp": 1625097600,
  "iam": {
    "roles": ["trader", "viewer"],
    "permissions": ["order:create", "portfolio:read"]
  }
}
```

**Service-to-Service Security:**

- **mTLS**: Mutual TLS for gRPC communication
- **Service Accounts**: Kubernetes-native identity
- **RBAC**: Role-based access control at K8s level
- **Network Policies**: Restrict pod-to-pod communication

### Security Testing

- **SAST**: Static application security testing
- **DAST**: Dynamic application security testing
- **Container Scanning**: Vulnerability scanning for Docker images
- **Dependency Scanning**: Check for vulnerable dependencies

## Monitoring & Observability

### Three Pillars of Observability

**Metrics (Prometheus + Grafana):**

```yaml
# Service metrics
http_requests_total{service="trading", method="POST", status="200"}
grpc_requests_duration_seconds{service="risk-calculator"}
```

**Logging (ELK Stack):**

```json
{
  "timestamp": "2025-06-20T10:00:00Z",
  "level": "INFO",
  "service": "trading-service",
  "traceId": "abc123",
  "spanId": "def456",
  "message": "Order created successfully"
}
```

**Tracing (Jaeger):**

- **Distributed tracing**: Track requests across services
- **Correlation IDs**: Link related operations
- **Performance monitoring**: Identify bottlenecks

### Application Performance Monitoring

- **Service dashboards**: Key business metrics
- **Error tracking**: Exception monitoring and alerting
- **SLA monitoring**: Track service level objectives

## Development Workflow

### Git Workflow

1. **Feature Branches**: `feature/service-name/feature-description`
2. **Pull Request Process**: Code review, automated testing
3. **Semantic Versioning**: Version services independently
4. **Monorepo Strategy**: All services in one repository

### CI/CD Pipeline

**GitHub Actions/GitLab CI Structure:**

```yaml
stages:
  - test
  - security-scan
  - build
  - deploy

test:
  script:
    - run unit tests
    - run integration tests
    - generate test coverage

security:
  script:
    - container security scan
    - dependency vulnerability check

deploy:
  script:
    - deploy to staging
    - run e2e tests
    - deploy to production (manual approval)
```

### Code Quality Gates

- **Test Coverage**: Minimum 80% for critical services
- **Security Scan**: No high/critical vulnerabilities
- **Performance**: Response time SLA compliance
- **Code Review**: At least one approval required

### Documentation Standards

- **ADRs**: Architecture Decision Records
- **API Documentation**: OpenAPI/gRPC docs
- **Runbooks**: Operational procedures
- **Troubleshooting Guides**: Common issues and solutions

## Coding Standards

### Rust Standards

- Use `rustfmt` for consistent formatting
- Enable all Clippy lints for code quality
- Follow Rust API guidelines
- Use `#[derive(Debug, Clone, PartialEq)]` appropriately

### Python Standards

- Follow PEP 8 style guide
- Use `black` for code formatting
- Use `mypy` for type checking
- Document functions with docstrings

### Java Standards

- Follow Google Java Style Guide
- Use `spotless` for code formatting
- Enable static analysis with SpotBugs
- Use Lombok judiciously

### TypeScript/Angular Standards

- Use Angular style guide
- Enable strict TypeScript compilation
- Use ESLint with recommended rules
- Implement proper error handling

---

**Remember**: These guidelines are living documents. Regularly review and update based on team feedback, technology evolution, and lessons learned from production deployments.
