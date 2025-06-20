# Microservices Development Guidelines

## Table of Contents

1. [Project Structure & Setup](#project-structure--setup)
2. [Architecture Principles](#architecture-principles)
3. [Technology Stack Standards](#technology-stack-standards)
4. [Coding Standards](#coding-standards)
5. [API Design Guidelines](#api-design-guidelines)
6. [Event Sourcing & CQRS Implementation](#event-sourcing--cqrs-implementation)
7. [Testing Strategy](#testing-strategy)
8. [Infrastructure as Code](#infrastructure-as-code)
9. [Security Guidelines](#security-guidelines)
10. [Monitoring & Observability](#monitoring--observability)

## Project Structure & Setup

### IntelliJ Multi-Module Structure

```
parent-project/
├── build.gradle.kts (root)
├── settings.gradle.kts
├── modules/
│   ├── rust-services/
│   │   ├── trading-engine/
│   │   ├── risk-management/
│   │   └── market-data/
│   ├── java-services/
│   │   ├── user-service/
│   │   ├── order-service/
│   │   └── notification-service/
│   ├── python-services/
│   │   ├── ml-prediction/
│   │   ├── analytics/
│   │   └── data-processing/
│   └── frontends/
│       ├── web-angular/
│       ├── mobile-android/
│       └── mobile-ios/
├── shared/
│   ├── schemas/
│   ├── proto/
│   └── common-libs/
├── infrastructure/
│   ├── terraform/
│   ├── kubernetes/
│   └── docker/
└── docs/
    ├── api/
    └── architecture/
```

### Module Configuration

- Each service as separate IntelliJ module with independent build configuration
- Shared dependencies in root `gradle.properties`
- Service-specific configurations in respective `build.gradle.kts`
- Use Gradle composite builds for multi-language support

## Architecture Principles

### 1. API-First Development

- **Protocol Buffers (gRPC)** for internal service communication
- **OpenAPI 3.0** specification before implementation
- **REST API Gateway** for external client communication
- **Contract-first** development approach

### 2. Event-Driven Architecture

- Event sourcing persists business entity state as sequence of state-changing events, with no update or delete operations
- **Domain Events** for cross-service communication
- **Event Store** as single source of truth
- **Asynchronous messaging** patterns

### 3. CQRS Pattern Implementation

- Separate read and write models with queryable replicas kept up to date by subscribing to events
- **Command Side**: Write operations only
- **Query Side**: Read-optimized projections
- **No direct updates or deletes** in business logic

### 4. Communication Patterns

- gRPC for internal microservices communication delivering higher throughput and lower latency
- **REST via API Gateway** for external clients
- **Message queues** for asynchronous communication
- **Circuit breaker** patterns for resilience

## Technology Stack Standards

### Rust Services (Actix-Web + RustQuant)

**Performance-critical services**: Trading engine, risk management, market data

#### Dependencies

```toml
[dependencies]
actix-web = "4.8"
tokio = { version = "1.0", features = ["full"] }
serde = { version = "1.0", features = ["derive"] }
serde_json = "1.0"
sqlx = { version = "0.8", features = ["postgres", "runtime-tokio-rustls"] }
uuid = { version = "1.0", features = ["v4", "serde"] }
chrono = { version = "0.4", features = ["serde"] }
tracing = "0.1"
tracing-subscriber = "0.3"
anyhow = "1.0"
thiserror = "1.0"
rustquant = "0.1"
tonic = "0.12" # gRPC
prost = "0.13"
```

### Java Services (Spring Boot)

**Business logic services**: User management, orders, notifications

#### Dependencies (build.gradle.kts)

```kotlin
dependencies {
    implementation("org.springframework.boot:spring-boot-starter-web")
    implementation("org.springframework.boot:spring-boot-starter-webflux")
    implementation("org.springframework.boot:spring-boot-starter-data-jpa")
    implementation("org.springframework.boot:spring-boot-starter-security")
    implementation("org.springframework.boot:spring-boot-starter-actuator")
    implementation("org.springframework.cloud:spring-cloud-starter-gateway")
    implementation("net.devh:grpc-spring-boot-starter:3.1.0")
    implementation("io.micrometer:micrometer-registry-prometheus")
    implementation("org.springframework.kafka:spring-kafka")
    testImplementation("org.springframework.boot:spring-boot-starter-test")
    testImplementation("org.testcontainers:junit-jupiter")
    testImplementation("org.testcontainers:postgresql")
}
```

### Python Services (JAX + Flax + Polars + Norse)

**ML/Analytics services**: Prediction models, data analytics, neural networks

#### Requirements (pyproject.toml)

```toml
[tool.poetry.dependencies]
python = "^3.11"
jax = "^0.4.25"
flax = "^0.8.2"
polars = "^0.20.15"
norse = "^1.0.0"
fastapi = "^0.110.0"
uvicorn = "^0.29.0"
pydantic = "^2.6.4"
grpcio = "^1.62.1"
grpcio-tools = "^1.62.1"
prometheus-client = "^0.20.0"
structlog = "^24.1.0"
```

### Frontend Technologies

#### Angular (TypeScript)

```json
{
  "dependencies": {
    "@angular/core": "^17.3.0",
    "@angular/common": "^17.3.0",
    "@angular/router": "^17.3.0",
    "@angular/forms": "^17.3.0",
    "@angular/material": "^17.3.0",
    "rxjs": "^7.8.0",
    "@ngrx/store": "^17.2.0",
    "@ngrx/effects": "^17.2.0"
  }
}
```

## Coding Standards

### Rust Coding Standards

#### Naming Conventions

- **Snake_case** for variables, functions, modules: `user_service`, `calculate_risk`
- **PascalCase** for types, structs, enums: `UserAccount`, `OrderStatus`
- **SCREAMING_SNAKE_CASE** for constants: `MAX_RETRY_ATTEMPTS`
- **kebab-case** for crate names: `trading-engine`

#### Code Structure

```rust
// File structure: lib.rs, main.rs, mod.rs pattern
// Use explicit visibility modifiers
pub struct TradingEngine {
    orders: HashMap<OrderId, Order>,
    risk_calculator: Arc<RiskCalculator>,
}

impl TradingEngine {
    pub fn new(config: &Config) -> Result<Self, EngineError> {
        // Constructor logic
    }

    pub async fn process_order(&self, order: Order) -> Result<OrderResult, EngineError> {
        // Business logic with proper error handling
    }
}

// Use Result<T, E> for error handling, avoid unwrap() in production
// Prefer ? operator over match for error propagation
// Use #[derive(Debug, Clone, Serialize, Deserialize)] appropriately
```

#### Error Handling

```rust
use thiserror::Error;

#[derive(Error, Debug)]
pub enum TradingError {
    #[error("Insufficient funds: required {required}, available {available}")]
    InsufficientFunds { required: Decimal, available: Decimal },

    #[error("Invalid order: {reason}")]
    InvalidOrder { reason: String },

    #[error("Database error: {0}")]
    Database(#[from] sqlx::Error),
}
```

### Java Coding Standards (Spring Boot)

#### Naming Conventions

- **camelCase** for variables, methods: `userService`, `calculateInterest`
- **PascalCase** for classes, interfaces: `UserService`, `OrderRepository`
- **CONSTANT_CASE** for constants: `MAX_RETRY_ATTEMPTS`
- **kebab-case** for property keys: `server.port`

#### Class Structure

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

    @GetMapping("/{id}")
    public ResponseEntity<UserDto> getUser(
            @PathVariable @NotNull UUID id,
            @RequestHeader("X-Request-ID") String requestId) {

        log.info("Fetching user with ID: {} for request: {}", id, requestId);

        return userService.findById(id)
            .map(user -> ResponseEntity.ok(userMapper.toDto(user)))
            .orElse(ResponseEntity.notFound().build());
    }
}
```

#### Service Layer Pattern

```java
@Service
@Transactional(readOnly = true)
public class UserService {

    private final UserRepository userRepository;
    private final UserEventPublisher eventPublisher;

    @Transactional
    public User createUser(CreateUserCommand command) {
        // Validation
        validateUserCreation(command);

        // Business logic
        User user = User.builder()
            .id(UUID.randomUUID())
            .email(command.getEmail())
            .createdAt(Instant.now())
            .build();

        User savedUser = userRepository.save(user);

        // Event publishing
        eventPublisher.publishUserCreated(savedUser);

        return savedUser;
    }
}
```

### Python Coding Standards (PEP 8 + Additional)

#### Naming Conventions

- **snake_case** for variables, functions, modules: `user_service`, `calculate_prediction`
- **PascalCase** for classes: `MLPredictor`, `DataProcessor`
- **CONSTANT_CASE** for constants: `MAX_BATCH_SIZE`

#### Code Structure

```python
from typing import Protocol, Optional, List
from dataclasses import dataclass
from abc import ABC, abstractmethod
import structlog

logger = structlog.get_logger()

@dataclass(frozen=True)
class PredictionRequest:
    """Immutable prediction request data."""
    features: List[float]
    model_version: str
    request_id: str

class MLPredictor(ABC):
    """Abstract base class for ML predictors."""

    @abstractmethod
    async def predict(self, request: PredictionRequest) -> float:
        """Generate prediction from input features."""
        pass

class JAXPredictor(MLPredictor):
    """JAX-based prediction implementation."""

    def __init__(self, model_path: str, config: dict):
        self.model = self._load_model(model_path)
        self.config = config

    async def predict(self, request: PredictionRequest) -> float:
        logger.info("Processing prediction", request_id=request.request_id)

        try:
            # Prediction logic using JAX
            prediction = self._run_inference(request.features)

            logger.info("Prediction completed",
                       request_id=request.request_id,
                       prediction=prediction)

            return prediction

        except Exception as e:
            logger.error("Prediction failed",
                        request_id=request.request_id,
                        error=str(e))
            raise
```

### TypeScript/Angular Standards

#### Naming Conventions

- **camelCase** for variables, functions: `userService`, `calculateTotal`
- **PascalCase** for classes, interfaces, types: `UserService`, `OrderData`
- **kebab-case** for component selectors: `app-user-profile`
- **CONSTANT_CASE** for constants: `API_BASE_URL`

#### Component Structure

```typescript
// user-profile.component.ts
@Component({
  selector: "app-user-profile",
  templateUrl: "./user-profile.component.html",
  styleUrls: ["./user-profile.component.scss"],
  changeDetection: ChangeDetectionStrategy.OnPush,
})
export class UserProfileComponent implements OnInit, OnDestroy {
  private readonly destroy$ = new Subject<void>();

  user$ = this.store.select(selectCurrentUser);
  loading$ = this.store.select(selectUserLoading);

  constructor(
    private readonly store: Store<AppState>,
    private readonly userService: UserService,
    private readonly cdr: ChangeDetectorRef
  ) {}

  ngOnInit(): void {
    this.store.dispatch(UserActions.loadUser());
  }

  ngOnDestroy(): void {
    this.destroy$.next();
    this.destroy$.complete();
  }

  onUpdateProfile(profileData: Partial<User>): void {
    this.store.dispatch(UserActions.updateUser({ profileData }));
  }
}
```

## API Design Guidelines

### gRPC Internal Communication

#### Proto Definition Standards

```protobuf
syntax = "proto3";

package trading.v1;

import "google/protobuf/timestamp.proto";
import "validate/validate.proto";

service TradingService {
  rpc CreateOrder(CreateOrderRequest) returns (CreateOrderResponse);
  rpc GetOrder(GetOrderRequest) returns (GetOrderResponse);
  rpc StreamMarketData(StreamMarketDataRequest) returns (stream MarketDataUpdate);
}

message CreateOrderRequest {
  string user_id = 1 [(validate.rules).string.uuid = true];
  string symbol = 2 [(validate.rules).string.min_len = 1];
  OrderType type = 3 [(validate.rules).enum.defined_only = true];
  double quantity = 4 [(validate.rules).double.gt = 0];
  double price = 5 [(validate.rules).double.gt = 0];
}

message Order {
  string id = 1;
  string user_id = 2;
  string symbol = 3;
  OrderType type = 4;
  OrderStatus status = 5;
  double quantity = 6;
  double price = 7;
  google.protobuf.Timestamp created_at = 8;
  google.protobuf.Timestamp updated_at = 9;
}

enum OrderType {
  ORDER_TYPE_UNSPECIFIED = 0;
  ORDER_TYPE_MARKET = 1;
  ORDER_TYPE_LIMIT = 2;
  ORDER_TYPE_STOP = 3;
}
```

### REST API Gateway (OpenAPI)

#### OpenAPI Specification Structure

```yaml
openapi: 3.0.3
info:
  title: Trading Platform API
  version: 1.0.0
  description: External API for trading platform
  contact:
    name: API Support
    email: api-support@company.com

servers:
  - url: https://api.trading-platform.com/v1
    description: Production server
  - url: https://staging-api.trading-platform.com/v1
    description: Staging server

security:
  - BearerAuth: []

paths:
  /orders:
    post:
      summary: Create new order
      operationId: createOrder
      tags: [Orders]
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: "#/components/schemas/CreateOrderRequest"
      responses:
        "201":
          description: Order created successfully
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/OrderResponse"
        "400":
          $ref: "#/components/responses/BadRequest"
        "401":
          $ref: "#/components/responses/Unauthorized"

components:
  schemas:
    CreateOrderRequest:
      type: object
      required: [symbol, type, quantity]
      properties:
        symbol:
          type: string
          pattern: "^[A-Z]{2,5}$"
          example: "AAPL"
        type:
          $ref: "#/components/schemas/OrderType"
        quantity:
          type: number
          minimum: 0.01
          example: 100.0
        price:
          type: number
          minimum: 0.01
          example: 150.25

    OrderType:
      type: string
      enum: [MARKET, LIMIT, STOP]

  securitySchemes:
    BearerAuth:
      type: http
      scheme: bearer
      bearerFormat: JWT
```

## Event Sourcing & CQRS Implementation

### Event Store Design

#### Event Schema

```rust
#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct DomainEvent {
    pub event_id: Uuid,
    pub aggregate_id: Uuid,
    pub aggregate_type: String,
    pub event_type: String,
    pub event_version: i32,
    pub event_data: serde_json::Value,
    pub metadata: EventMetadata,
    pub occurred_at: DateTime<Utc>,
}

#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct EventMetadata {
    pub user_id: Option<Uuid>,
    pub correlation_id: Uuid,
    pub causation_id: Option<Uuid>,
    pub command_id: Option<Uuid>,
}
```

#### Aggregate Root Pattern

```rust
pub trait AggregateRoot {
    type Event: DomainEvent;
    type Error: std::error::Error;

    fn apply_event(&mut self, event: &Self::Event) -> Result<(), Self::Error>;
    fn get_uncommitted_events(&self) -> &[Self::Event];
    fn mark_events_as_committed(&mut self);
}

#[derive(Debug)]
pub struct Order {
    id: OrderId,
    user_id: UserId,
    status: OrderStatus,
    // ... other fields
    uncommitted_events: Vec<OrderEvent>,
}

impl AggregateRoot for Order {
    type Event = OrderEvent;
    type Error = OrderError;

    fn apply_event(&mut self, event: &OrderEvent) -> Result<(), OrderError> {
        match event {
            OrderEvent::OrderCreated { order_id, user_id, .. } => {
                self.id = *order_id;
                self.user_id = *user_id;
                self.status = OrderStatus::Pending;
            },
            OrderEvent::OrderExecuted { .. } => {
                self.status = OrderStatus::Executed;
            },
            // Handle other events
        }
        Ok(())
    }
}
```

### CQRS Query Side

#### Read Model Projections

```java
@Component
@EventHandler
@Slf4j
public class OrderProjectionHandler {

    private final OrderQueryRepository repository;

    @EventHandlerMethod
    public void handle(OrderCreatedEvent event) {
        log.info("Projecting OrderCreated event: {}", event.getOrderId());

        OrderQueryModel queryModel = OrderQueryModel.builder()
            .id(event.getOrderId())
            .userId(event.getUserId())
            .symbol(event.getSymbol())
            .status("PENDING")
            .createdAt(event.getOccurredAt())
            .build();

        repository.save(queryModel);
    }

    @EventHandlerMethod
    public void handle(OrderExecutedEvent event) {
        repository.findById(event.getOrderId())
            .ifPresent(order -> {
                order.setStatus("EXECUTED");
                order.setExecutedAt(event.getOccurredAt());
                repository.save(order);
            });
    }
}
```

## Testing Strategy

### Unit Testing

#### Rust Unit Tests

```rust
#[cfg(test)]
mod tests {
    use super::*;
    use tokio_test;

    #[tokio::test]
    async fn test_order_creation_with_valid_data() {
        // Arrange
        let order_data = OrderData {
            user_id: Uuid::new_v4(),
            symbol: "AAPL".to_string(),
            quantity: Decimal::from(100),
            price: Decimal::from_str("150.25").unwrap(),
        };

        // Act
        let result = Order::create(order_data).await;

        // Assert
        assert!(result.is_ok());
        let order = result.unwrap();
        assert_eq!(order.status(), OrderStatus::Pending);
        assert_eq!(order.uncommitted_events().len(), 1);
    }

    #[tokio::test]
    async fn test_order_creation_with_invalid_quantity() {
        // Arrange
        let order_data = OrderData {
            quantity: Decimal::ZERO,
            // ... other fields
        };

        // Act & Assert
        let result = Order::create(order_data).await;
        assert!(matches!(result, Err(OrderError::InvalidQuantity)));
    }
}
```

#### Java Unit Tests (JUnit 5 + Mockito)

```java
@ExtendWith(MockitoExtension.class)
class UserServiceTest {

    @Mock
    private UserRepository userRepository;

    @Mock
    private UserEventPublisher eventPublisher;

    @InjectMocks
    private UserService userService;

    @Test
    @DisplayName("Should create user successfully with valid data")
    void shouldCreateUserSuccessfully() {
        // Given
        CreateUserCommand command = CreateUserCommand.builder()
            .email("test@example.com")
            .firstName("John")
            .lastName("Doe")
            .build();

        User expectedUser = User.builder()
            .email(command.getEmail())
            .firstName(command.getFirstName())
            .lastName(command.getLastName())
            .build();

        when(userRepository.save(any(User.class))).thenReturn(expectedUser);

        // When
        User result = userService.createUser(command);

        // Then
        assertThat(result.getEmail()).isEqualTo("test@example.com");
        verify(userRepository).save(any(User.class));
        verify(eventPublisher).publishUserCreated(any(User.class));
    }

    @Test
    @DisplayName("Should throw exception when email already exists")
    void shouldThrowExceptionWhenEmailExists() {
        // Given
        CreateUserCommand command = CreateUserCommand.builder()
            .email("existing@example.com")
            .build();

        when(userRepository.existsByEmail(command.getEmail())).thenReturn(true);

        // When & Then
        assertThatThrownBy(() -> userService.createUser(command))
            .isInstanceOf(UserAlreadyExistsException.class)
            .hasMessage("User with email existing@example.com already exists");
    }
}
```

### Integration Testing

#### Testcontainers for Java

```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@TestPropertySource(properties = {
    "spring.datasource.url=jdbc:tc:postgresql:15:///testdb",
    "spring.kafka.bootstrap-servers=${embedded.kafka.brokers}"
})
class UserControllerIntegrationTest {

    @Autowired
    private TestRestTemplate restTemplate;

    @Autowired
    private UserRepository userRepository;

    @Container
    static PostgreSQLContainer<?> postgresql = new PostgreSQLContainer<>("postgres:15")
            .withDatabaseName("testdb")
            .withUsername("test")
            .withPassword("test");

    @Test
    void shouldCreateUserViaAPI() {
        // Given
        CreateUserRequest request = CreateUserRequest.builder()
            .email("test@example.com")
            .firstName("John")
            .lastName("Doe")
            .build();

        // When
        ResponseEntity<UserResponse> response = restTemplate.postForEntity(
            "/api/v1/users",
            request,
            UserResponse.class
        );

        // Then
        assertThat(response.getStatusCode()).isEqualTo(HttpStatus.CREATED);
        assertThat(response.getBody().getEmail()).isEqualTo("test@example.com");

        // Verify database state
        Optional<User> savedUser = userRepository.findByEmail("test@example.com");
        assertThat(savedUser).isPresent();
    }
}
```

### End-to-End Testing

#### Cypress for Angular

```typescript
// cypress/e2e/user-profile.cy.ts
describe("User Profile Management", () => {
  beforeEach(() => {
    cy.login("test@example.com", "password");
    cy.visit("/profile");
  });

  it("should display user profile information", () => {
    cy.get("[data-cy=user-name]").should("contain", "John Doe");
    cy.get("[data-cy=user-email]").should("contain", "test@example.com");
  });

  it("should update user profile successfully", () => {
    cy.get("[data-cy=edit-profile-btn]").click();
    cy.get("[data-cy=first-name-input]").clear().type("Jane");
    cy.get("[data-cy=save-profile-btn]").click();

    cy.get("[data-cy=success-message]").should("be.visible");
    cy.get("[data-cy=user-name]").should("contain", "Jane Doe");
  });
});
```

### Performance Testing

#### Artillery.js Configuration

```yaml
# performance-tests/load-test.yml
config:
  target: "https://api.trading-platform.com"
  phases:
    - duration: 60
      arrivalRate: 10
      name: "Warm up"
    - duration: 300
      arrivalRate: 50
      name: "Sustained load"
    - duration: 60
      arrivalRate: 100
      name: "Peak load"

scenarios:
  - name: "Create and retrieve orders"
    weight: 70
    flow:
      - post:
          url: "/v1/orders"
          headers:
            Authorization: "Bearer {{ $randomString() }}"
          json:
            symbol: "AAPL"
            type: "LIMIT"
            quantity: 100
            price: 150.25
          capture:
            - json: "$.id"
              as: "orderId"
      - get:
          url: "/v1/orders/{{ orderId }}"
          headers:
            Authorization: "Bearer {{ $randomString() }}"

  - name: "Market data streaming"
    weight: 30
    flow:
      - get:
          url: "/v1/market-data/stream"
          headers:
            Authorization: "Bearer {{ $randomString() }}"
```

## Infrastructure as Code

### Terraform Configuration

#### Main Infrastructure

```hcl
# infrastructure/terraform/main.tf
terraform {
  required_version = ">= 1.0"
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
    kubernetes = {
      source  = "hashicorp/kubernetes"
      version = "~> 2.20"
    }
  }

  backend "s3" {
    bucket = "terraform-state-trading-platform"
    key    = "infrastructure/terraform.tfstate"
    region = "us-west-2"
  }
}

module "vpc" {
  source = "./modules/vpc"

  name = var.environment
  cidr = var.vpc_cidr

  availability_zones = var.availability_zones
  private_subnets    = var.private_subnets
  public_subnets     = var.public_subnets

  enable_nat_gateway = true
  enable_vpn_gateway = false

  tags = local.common_tags
}

module "eks" {
  source = "./modules/eks"

  cluster_name    = "${var.environment}-trading-cluster"
  cluster_version = "1.29"

  vpc_id     = module.vpc.vpc_id
  subnet_ids = module.vpc.private_subnets

  node_groups = {
    rust_services = {
      instance_types = ["c5.large", "c5.xlarge"]
      scaling_config = {
        desired_size = 3
        max_size     = 10
        min_size     = 1
      }
      k8s_labels = {
        workload = "rust"
      }
    }

    java_services = {
      instance_types = ["m5.large", "m5.xlarge"]
      scaling_config = {
        desired_size = 2
        max_size     = 8
        min_size     = 1
      }
      k8s_labels = {
        workload = "java"
      }
    }

    python_ml = {
      instance_types = ["p3.2xlarge"]
      scaling_config = {
        desired_size = 1
        max_size     = 3
        min_size     = 0
      }
      k8s_labels = {
        workload = "ml"
      }
    }
  }

  tags = local.common_tags
}
```

### Kubernetes Manifests

#### Service Deployment Template

```yaml
# infrastructure/kubernetes/templates/service-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Values.service.name }}
  namespace: {{ .Values.namespace }}
  labels:
    app: {{ .Values.service.name }}
    version: {{ .Values.service.version }}
spec:
  replicas: {{ .Values.service.replicas }}
  selector:
    matchLabels:
      app: {{ .Values.service.name }}
  template:
    metadata:
      labels:
        app: {{ .Values.service.name }}
        version: {{ .Values.service.version }}
      annotations:
        prometheus.io/scrape: "true"
        prometheus.io/port: "{{ .Values.service.metricsPort }}"
        prometheus.io/path: "/metrics"
    spec:
      containers:
      - name: {{ .Values.service.name }}
        image: {{ .Values.service.image }}:{{ .Values.service.version }}
        ports:
        - containerPort: {{ .Values.service.port }}
          name: http
        - containerPort: {{ .Values.service.grpcPort }}
          name: grpc
        - containerPort: {{ .Values.service.metricsPort }}
          name: metrics
        env:
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: {{ .Values.service.name }}-secrets
              key: database-url
        - name: KAFKA_BROKERS
          value: {{ .Values.kafka.brokers }}
        - name: JAEGER_ENDPOINT
          value: {{ .Values.jaeger.endpoint }}
        resources:
          requests:
            memory: {{ .Values.service.resources.requests.memory }}
            cpu: {{ .Values.service.resources.requests.cpu }}
          limits:
            memory: {{ .Values.service.resources.limits.memory }}
            cpu: {{ .Values.service.resources.limits.cpu }}
        livenessProbe:
          httpGet:
            path: /health
            port: http
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: http
          initialDelaySeconds: 5
          periodSeconds: 5
        securityContext:
          allowPrivilegeEscalation: false
          runAsNonRoot: true
          runAsUser: 1000
          capabilities:
            drop:
            - ALL
---
apiVersion: v1
kind: Service
metadata:
  name: {{ .Values.service.name }}
  namespace: {{ .Values.namespace }}
  labels:
    app: {{ .Values.service.name }}
spec:
  ports:
  - port: 80
    targetPort: http
    protocol: TCP
    name: http
  - port: {{ .Values.service.grpcPort }}
    targetPort: grpc
    protocol: TCP
    name: grpc
  selector:
    app: {{ .Values.service.name }}
```

### Docker Configuration

#### Multi-stage Rust Dockerfile

```dockerfile
# Dockerfile.rust
FROM rust:1.76-slim as builder

WORKDIR /app

# Install system dependencies
RUN apt-get update && apt-get install -y \
    pkg-config \
    libssl-dev \
    && rm -rf /var/lib/apt/lists/*

# Copy dependency files
COPY Cargo.toml Cargo.lock ./
COPY src ./src

# Build with optimizations
RUN cargo build --release

# Runtime image
FROM debian:bookworm-slim

RUN apt-get update && apt-get install -y \
    ca-certificates \
    && rm -rf /var/lib/apt/lists/*

# Create non-root user
RUN useradd -r -s /bin/false appuser

WORKDIR /app

# Copy binary from builder
COPY --from=builder /app/target/release/trading-service /app/

# Change ownership
RUN chown -R appuser:appuser /app
USER appuser

EXPOSE 8080 9090

CMD ["./trading-service"]
```

#### Java Spring Boot Dockerfile

```dockerfile
# Dockerfile.java
FROM eclipse-temurin:21-jdk-alpine as builder

WORKDIR /app

# Copy gradle files
COPY build.gradle.kts settings.gradle.kts gradlew ./
COPY gradle ./gradle

# Copy source code
COPY src ./src

# Build application
RUN ./gradlew build -x test

# Runtime image
FROM eclipse-temurin:21-jre-alpine

RUN addgroup -g 1000 appgroup && \
    adduser -u 1000 -G appgroup -s /bin/sh -D appuser

WORKDIR /app

# Copy JAR from builder
COPY --from=builder /app/build/libs/*.jar app.jar

RUN chown appuser:appgroup app.jar
USER appuser

EXPOSE 8080 9090

ENTRYPOINT ["java", "-jar", "app.jar"]
```

#### Python FastAPI Dockerfile

```dockerfile
# Dockerfile.python
FROM python:3.11-slim as builder

WORKDIR /app

# Install system dependencies
RUN apt-get update && apt-get install -y \
    build-essential \
    && rm -rf /var/lib/apt/lists/*

# Install Poetry
RUN pip install poetry

# Copy dependency files
COPY pyproject.toml poetry.lock ./

# Configure poetry
RUN poetry config virtualenvs.create false

# Install dependencies
RUN poetry install --only=main --no-dev

# Runtime image
FROM python:3.11-slim

RUN apt-get update && apt-get install -y \
    && rm -rf /var/lib/apt/lists/*

# Create non-root user
RUN useradd -r -s /bin/false appuser

WORKDIR /app

# Copy installed packages from builder
COPY --from=builder /usr/local/lib/python3.11/site-packages /usr/local/lib/python3.11/site-packages
COPY --from=builder /usr/local/bin /usr/local/bin

# Copy application code
COPY src ./src

RUN chown -R appuser:appuser /app
USER appuser

EXPOSE 8000 9090

CMD ["uvicorn", "src.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

## Security Guidelines

### Identity and Access Management (IAM)

#### Keycloak Configuration

```yaml
# infrastructure/kubernetes/keycloak/keycloak-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: keycloak
  namespace: auth
spec:
  replicas: 2
  selector:
    matchLabels:
      app: keycloak
  template:
    metadata:
      labels:
        app: keycloak
    spec:
      containers:
        - name: keycloak
          image: quay.io/keycloak/keycloak:24.0
          args:
            - start
            - --hostname-strict=false
            - --hostname-strict-https=false
            - --proxy=edge
          env:
            - name: KEYCLOAK_ADMIN
              value: admin
            - name: KEYCLOAK_ADMIN_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: keycloak-secrets
                  key: admin-password
            - name: KC_DB
              value: postgres
            - name: KC_DB_URL
              valueFrom:
                secretKeyRef:
                  name: keycloak-secrets
                  key: database-url
            - name: KC_DB_USERNAME
              valueFrom:
                secretKeyRef:
                  name: keycloak-secrets
                  key: database-username
            - name: KC_DB_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: keycloak-secrets
                  key: database-password
          ports:
            - containerPort: 8080
          resources:
            requests:
              memory: 512Mi
              cpu: 500m
            limits:
              memory: 1Gi
              cpu: 1000m
```

#### JWT Token Validation (Rust)

```rust
use jsonwebtoken::{decode, Algorithm, DecodingKey, Validation};
use serde::{Deserialize, Serialize};

#[derive(Debug, Serialize, Deserialize)]
pub struct Claims {
    pub sub: String,
    pub exp: usize,
    pub iat: usize,
    pub aud: String,
    pub iss: String,
    pub roles: Vec<String>,
    pub permissions: Vec<String>,
}

pub struct JwtValidator {
    decoding_key: DecodingKey,
    validation: Validation,
}

impl JwtValidator {
    pub fn new(jwks_url: &str) -> Result<Self, JwtError> {
        let jwks = fetch_jwks(jwks_url)?;
        let decoding_key = DecodingKey::from_jwk(&jwks)?;

        let mut validation = Validation::new(Algorithm::RS256);
        validation.set_audience(&["trading-platform"]);
        validation.set_issuer(&["https://auth.trading-platform.com"]);

        Ok(Self {
            decoding_key,
            validation,
        })
    }

    pub fn validate_token(&self, token: &str) -> Result<Claims, JwtError> {
        let token_data = decode::<Claims>(token, &self.decoding_key, &self.validation)?;
        Ok(token_data.claims)
    }

    pub fn has_permission(&self, claims: &Claims, required_permission: &str) -> bool {
        claims.permissions.contains(&required_permission.to_string())
    }
}

// Middleware for Actix-Web
pub async fn jwt_middleware(
    req: ServiceRequest,
    credentials: BearerAuth,
) -> Result<ServiceRequest, (Error, ServiceRequest)> {
    let validator = req.app_data::<JwtValidator>()
        .ok_or_else(|| (AuthError::ConfigurationError.into(), req))?;

    let claims = validator.validate_token(credentials.token())
        .map_err(|e| (AuthError::InvalidToken(e).into(), req))?;

    req.extensions_mut().insert(claims);
    Ok(req)
}
```

#### Spring Security Configuration (Java)

```java
@Configuration
@EnableWebSecurity
@EnableMethodSecurity
public class SecurityConfig {

    @Value("${spring.security.oauth2.resourceserver.jwt.jwk-set-uri}")
    private String jwkSetUri;

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(authz -> authz
                .requestMatchers("/actuator/health", "/actuator/info").permitAll()
                .requestMatchers("/api/v1/public/**").permitAll()
                .requestMatchers(HttpMethod.GET, "/api/v1/orders/**").hasRole("USER")
                .requestMatchers(HttpMethod.POST, "/api/v1/orders/**").hasRole("TRADER")
                .requestMatchers("/api/v1/admin/**").hasRole("ADMIN")
                .anyRequest().authenticated()
            )
            .oauth2ResourceServer(oauth2 -> oauth2
                .jwt(jwt -> jwt
                    .jwkSetUri(jwkSetUri)
                    .jwtAuthenticationConverter(jwtAuthenticationConverter())
                )
            )
            .sessionManagement(session -> session
                .sessionCreationPolicy(SessionCreationPolicy.STATELESS)
            )
            .csrf(csrf -> csrf.disable())
            .cors(cors -> cors.configurationSource(corsConfigurationSource()));

        return http.build();
    }

    @Bean
    public JwtAuthenticationConverter jwtAuthenticationConverter() {
        JwtGrantedAuthoritiesConverter authoritiesConverter =
            new JwtGrantedAuthoritiesConverter();
        authoritiesConverter.setAuthorityPrefix("ROLE_");
        authoritiesConverter.setAuthoritiesClaimName("roles");

        JwtAuthenticationConverter converter = new JwtAuthenticationConverter();
        converter.setJwtGrantedAuthoritiesConverter(authoritiesConverter);
        return converter;
    }
}
```

### API Security Best Practices

#### Rate Limiting Configuration

```yaml
# infrastructure/kubernetes/istio/rate-limit.yaml
apiVersion: networking.istio.io/v1alpha3
kind: EnvoyFilter
metadata:
  name: rate-limit-filter
  namespace: istio-system
spec:
  configPatches:
    - applyTo: HTTP_FILTER
      match:
        context: SIDECAR_INBOUND
        listener:
          filterChain:
            filter:
              name: "envoy.filters.network.http_connection_manager"
      patch:
        operation: INSERT_BEFORE
        value:
          name: envoy.filters.http.local_ratelimit
          typed_config:
            "@type": type.googleapis.com/udpa.type.v1.TypedStruct
            type_url: type.googleapis.com/envoy.extensions.filters.http.local_ratelimit.v3.LocalRateLimit
            value:
              stat_prefix: rate_limiter
              token_bucket:
                max_tokens: 100
                tokens_per_fill: 100
                fill_interval: 60s
              filter_enabled:
                runtime_key: rate_limit_enabled
                default_value:
                  numerator: 100
                  denominator: HUNDRED
              filter_enforced:
                runtime_key: rate_limit_enforced
                default_value:
                  numerator: 100
                  denominator: HUNDRED
```

## Monitoring & Observability

### Prometheus Metrics

#### Rust Metrics Implementation

```rust
use prometheus::{
    Counter, Histogram, IntCounter, IntGauge, Registry, TextEncoder,
    HistogramOpts, Opts,
};
use std::sync::Arc;

#[derive(Clone)]
pub struct Metrics {
    pub requests_total: Counter,
    pub request_duration: Histogram,
    pub active_connections: IntGauge,
    pub orders_processed: IntCounter,
    pub errors_total: Counter,
}

impl Metrics {
    pub fn new() -> Result<Self, prometheus::Error> {
        let requests_total = Counter::with_opts(Opts::new(
            "http_requests_total",
            "Total number of HTTP requests"
        ).const_labels(prometheus::labels! {
            "service" => "trading-engine",
        }))?;

        let request_duration = Histogram::with_opts(HistogramOpts::new(
            "http_request_duration_seconds",
            "HTTP request duration in seconds"
        ).buckets(vec![0.001, 0.005, 0.01, 0.05, 0.1, 0.5, 1.0, 5.0]))?;

        let active_connections = IntGauge::with_opts(Opts::new(
            "active_connections",
            "Number of active connections"
        ))?;

        let orders_processed = IntCounter::with_opts(Opts::new(
            "orders_processed_total",
            "Total number of orders processed"
        ))?;

        let errors_total = Counter::with_opts(Opts::new(
            "errors_total",
            "Total number of errors"
        ))?;

        Ok(Self {
            requests_total,
            request_duration,
            active_connections,
            orders_processed,
            errors_total,
        })
    }

    pub fn register(&self, registry: &Registry) -> Result<(), prometheus::Error> {
        registry.register(Box::new(self.requests_total.clone()))?;
        registry.register(Box::new(self.request_duration.clone()))?;
        registry.register(Box::new(self.active_connections.clone()))?;
        registry.register(Box::new(self.orders_processed.clone()))?;
        registry.register(Box::new(self.errors_total.clone()))?;
        Ok(())
    }
}

// Middleware for metrics collection
pub async fn metrics_middleware(
    req: ServiceRequest,
    srv: &mut dyn Service<ServiceRequest, Response = ServiceResponse, Error = Error>,
) -> Result<ServiceResponse, Error> {
    let start_time = Instant::now();
    let metrics = req.app_data::<Arc<Metrics>>().unwrap();

    metrics.requests_total.inc();
    metrics.active_connections.inc();

    let response = srv.call(req).await;

    let duration = start_time.elapsed().as_secs_f64();
    metrics.request_duration.observe(duration);
    metrics.active_connections.dec();

    if let Ok(ref res) = response {
        if res.status().is_server_error() {
            metrics.errors_total.inc();
        }
    }

    response
}
```

#### Java Micrometer Metrics

```java
@Component
public class TradingMetrics {

    private final Counter ordersCreated;
    private final Counter ordersExecuted;
    private final Timer orderProcessingTime;
    private final Gauge activeOrders;

    public TradingMetrics(MeterRegistry meterRegistry) {
        this.ordersCreated = Counter.builder("orders.created")
            .description("Total number of orders created")
            .tag("service", "order-service")
            .register(meterRegistry);

        this.ordersExecuted = Counter.builder("orders.executed")
            .description("Total number of orders executed")
            .tag("service", "order-service")
            .register(meterRegistry);

        this.orderProcessingTime = Timer.builder("order.processing.time")
            .description("Time taken to process an order")
            .tag("service", "order-service")
            .register(meterRegistry);

        this.activeOrders = Gauge.builder("orders.active")
            .description("Number of active orders")
            .tag("service", "order-service")
            .register(meterRegistry, this, TradingMetrics::getActiveOrderCount);
    }

    public void incrementOrdersCreated() {
        ordersCreated.increment();
    }

    public void recordOrderProcessingTime(Duration duration) {
        orderProcessingTime.record(duration);
    }

    private double getActiveOrderCount() {
        // Implementation to get active order count
        return orderRepository.countByStatus(OrderStatus.ACTIVE);
    }
}

@RestController
@Timed(value = "api.requests", description = "API request duration")
public class OrderController {

    private final TradingMetrics metrics;

    @PostMapping("/orders")
    @Counted(value = "orders.api.created", description = "Orders created via API")
    public ResponseEntity<OrderResponse> createOrder(@RequestBody CreateOrderRequest request) {
        Timer.Sample sample = Timer.start();

        try {
            Order order = orderService.createOrder(request);
            metrics.incrementOrdersCreated();

            return ResponseEntity.ok(orderMapper.toResponse(order));
        } finally {
            sample.stop(metrics.getOrderProcessingTime());
        }
    }
}
```

### Distributed Tracing

#### Jaeger Configuration

```yaml
# infrastructure/kubernetes/jaeger/jaeger-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: jaeger
  namespace: observability
spec:
  replicas: 1
  selector:
    matchLabels:
      app: jaeger
  template:
    metadata:
      labels:
        app: jaeger
    spec:
      containers:
        - name: jaeger
          image: jaegertracing/all-in-one:1.52
          env:
            - name: COLLECTOR_ZIPKIN_HOST_PORT
              value: ":9411"
            - name: COLLECTOR_OTLP_ENABLED
              value: "true"
          ports:
            - containerPort: 16686
              name: ui
            - containerPort: 14268
              name: collector
            - containerPort: 14250
              name: grpc
            - containerPort: 9411
              name: zipkin
          resources:
            requests:
              memory: 256Mi
              cpu: 100m
            limits:
              memory: 512Mi
              cpu: 200m
```

#### OpenTelemetry Rust Implementation

```rust
use opentelemetry::{
    global, sdk::propagation::TraceContextPropagator, trace::TraceError,
};
use opentelemetry_jaeger::JaegerPipeline;
use tracing_subscriber::{layer::SubscriberExt, util::SubscriberInitExt};

pub fn init_tracing() -> Result<(), TraceError> {
    global::set_text_map_propagator(TraceContextPropagator::new());

    let tracer = JaegerPipeline::new()
        .with_service_name("trading-engine")
        .with_service_version("1.0.0")
        .with_endpoint("http://jaeger:14268/api/traces")
        .install_batch(opentelemetry::runtime::Tokio)?;

    let telemetry = tracing_opentelemetry::layer().with_tracer(tracer);

    tracing_subscriber::registry()
        .with(telemetry)
        .with(tracing_subscriber::EnvFilter::from_default_env())
        .with(tracing_subscriber::fmt::layer())
        .init();

    Ok(())
}

// Usage in service methods
#[tracing::instrument(skip(self), fields(order_id = %order_id))]
pub async fn process_order(&self, order_id: OrderId) -> Result<Order, OrderError> {
    tracing::info!("Processing order");

    let span = tracing::Span::current();
    span.set_attribute("order.symbol", order.symbol.clone());
    span.set_attribute("order.quantity", order.quantity.to_string());

    // Business logic
    let result = self.validate_order(&order).await?;

    tracing::info!("Order processed successfully");
    Ok(result)
}
```

### Logging Standards

#### Structured Logging (Rust)

```rust
use serde::Serialize;
use tracing::{info, warn, error};

#[derive(Serialize)]
struct OrderProcessedEvent {
    order_id: String,
    user_id: String,
    symbol: String,
    quantity: f64,
    price: f64,
    processing_time_ms: u64,
}

impl OrderService {
    #[tracing::instrument(skip(self))]
    pub async fn process_order(&self, order: Order) -> Result<OrderResult, OrderError> {
        let start_time = Instant::now();

        info!(
            order_id = %order.id,
            user_id = %order.user_id,
            symbol = %order.symbol,
            "Starting order processing"
        );

        match self.validate_and_execute_order(order).await {
            Ok(result) => {
                let processing_time = start_time.elapsed().as_millis() as u64;

                let event = OrderProcessedEvent {
                    order_id: order.id.to_string(),
                    user_id: order.user_id.to_string(),
                    symbol: order.symbol.clone(),
                    quantity: order.quantity.to_f64(),
                    price: order.price.to_f64(),
                    processing_time_ms: processing_time,
                };

                info!(
                    %event,
                    processing_time_ms = processing_time,
                    "Order processed successfully"
                );

                Ok(result)
            }
            Err(e) => {
                error!(
                    order_id = %order.id,
                    error = %e,
                    "Failed to process order"
                );
                Err(e)
            }
        }
    }
}
```

## Deployment & CI/CD

### GitHub Actions Pipeline

#### Multi-Service CI/CD

```yaml
# .github/workflows/ci-cd.yml
name: CI/CD Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

env:
  REGISTRY: ghcr.io
  IMAGE_NAME_PREFIX: ${{ github.repository }}

jobs:
  detect-changes:
    runs-on: ubuntu-latest
    outputs:
      rust-services: ${{ steps.changes.outputs.rust-services }}
      java-services: ${{ steps.changes.outputs.java-services }}
      python-services: ${{ steps.changes.outputs.python-services }}
      angular-frontend: ${{ steps.changes.outputs.angular-frontend }}
    steps:
      - uses: actions/checkout@v4
      - uses: dorny/paths-filter@v2
        id: changes
        with:
          filters: |
            rust-services:
              - 'modules/rust-services/**'
            java-services:
              - 'modules/java-services/**'
            python-services:
              - 'modules/python-services/**'
            angular-frontend:
              - 'modules/frontends/web-angular/**'

  test-rust-services:
    needs: detect-changes
    if: needs.detect-changes.outputs.rust-services == 'true'
    runs-on: ubuntu-latest
    strategy:
      matrix:
        service: [trading-engine, risk-management, market-data]
    steps:
      - uses: actions/checkout@v4
      - uses: actions-rs/toolchain@v1
        with:
          toolchain: stable
          override: true
          components: rustfmt, clippy

      - name: Cache cargo registry
        uses: actions/cache@v3
        with:
          path: ~/.cargo/registry
          key: ${{ runner.os }}-cargo-registry-${{ hashFiles('**/Cargo.lock') }}

      - name: Run tests
        working-directory: modules/rust-services/${{ matrix.service }}
        run: |
          cargo test --verbose
          cargo clippy -- -D warnings
          cargo fmt -- --check

  build-rust-services:
    needs: [detect-changes, test-rust-services]
    if: needs.detect-changes.outputs.rust-services == 'true'
    runs-on: ubuntu-latest
    strategy:
      matrix:
        service: [trading-engine, risk-management, market-data]
    steps:
      - uses: actions/checkout@v4
      - name: Log in to Container Registry
        uses: docker/login-action@v2
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Build and push Docker image
        uses: docker/build-push-action@v4
        with:
          context: modules/rust-services/${{ matrix.service }}
          file: modules/rust-services/${{ matrix.service }}/Dockerfile
          push: true
          tags: |
            ${{ env.REGISTRY }}/${{ env.IMAGE_NAME_PREFIX }}/rust-${{ matrix.service }}:${{ github.sha }}
            ${{ env.REGISTRY }}/${{ env.IMAGE_NAME_PREFIX }}/rust-${{ matrix.service }}:latest

  deploy-staging:
    needs: [build-rust-services, build-java-services, build-python-services]
    if: github.ref == 'refs/heads/develop'
    runs-on: ubuntu-latest
    environment: staging
    steps:
      - uses: actions/checkout@v4
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v2
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: us-west-2

      - name: Deploy to EKS
        run: |
          aws eks update-kubeconfig --name staging-trading-cluster
          helm upgrade --install trading-platform ./infrastructure/helm/trading-platform \
            --namespace trading-platform \
            --set image.tag=${{ github.sha }} \
            --set environment=staging \
            --wait --timeout 10m

  security-scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Run Trivy vulnerability scanner
        uses: aquasecurity/trivy-action@master
        with:
          scan-type: "fs"
          scan-ref: "."
          format: "sarif"
          output: "trivy-results.sarif"

      - name: Upload Trivy scan results
        uses: github/codeql-action/upload-sarif@v2
        with:
          sarif_file: "trivy-results.sarif"
```

## Development Workflow & Best Practices

### Git Workflow

- **Main Branch**: Production-ready code
- **Develop Branch**: Integration branch for features
- **Feature Branches**: `feature/TICKET-ID-description`
- **Hotfix Branches**: `hotfix/TICKET-ID-description`
- **Release Branches**: `release/v1.2.3`

### Code Review Guidelines

1. **Automated Checks**: All tests pass, code formatting verified
2. **Security Review**: Authentication, authorization, input validation
3. **Performance Review**: Database queries, caching, resource usage
4. **Architecture Review**: CQRS compliance, event sourcing patterns
5. **Documentation**: API changes documented, README updates

### Environment Strategy

- **Development**: Local development with Docker Compose
- **Staging**: Kubernetes cluster with production-like configuration
- **Production**: Multi-region Kubernetes deployment with blue-green deployments

### Data Management

- **Event Store**: PostgreSQL with event sourcing tables
- **Read Models**: Dedicated databases per service (PostgreSQL, MongoDB)
- **Caching**: Redis for distributed caching
- **Message Queue**: Apache Kafka for event streaming

This comprehensive guide ensures consistent development practices across all services in your microservices architecture while maintaining high standards for code quality, security, and observability.
