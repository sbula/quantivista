# Order Management Service

## Responsibility
Complete order lifecycle management with event sourcing. Handles order creation, modification, cancellation, and tracking throughout the entire execution process with comprehensive audit trails.

## Technology Stack
- **Language**: Java + Spring Boot + Event Sourcing + PostgreSQL
- **Libraries**: Spring Boot, Axon Framework, PostgreSQL, Redis
- **Data Processing**: Polars for high-performance data manipulation (5-10x faster than pandas)
- **Analytics**: DuckDB for complex analytical queries and aggregations
- **ML Framework**: JAX for custom optimization algorithms and advanced models
- **Scaling**: Horizontal by order volume, event-driven architecture
- **NFRs**: P99 order processing < 50ms, 99.99% order integrity, complete audit trail

## API Specification

### Core APIs
```pseudo
// Enumerations
enum OrderType {
    MARKET,
    LIMIT,
    STOP,
    STOP_LIMIT,
    ICEBERG,
    TWAP,
    VWAP
}

enum OrderStatus {
    PENDING,
    SUBMITTED,
    PARTIALLY_FILLED,
    FILLED,
    CANCELLED,
    REJECTED,
    EXPIRED
}

enum OrderSide {
    BUY,
    SELL
}

// Data Models
struct OrderCreationRequest {
    order_id: String
    instrument_id: String
    side: OrderSide
    order_type: OrderType
    quantity: Float
    price: Optional<Float>
    time_in_force: String
    execution_instructions: ExecutionInstructions
}

struct Order {
    order_id: String
    instrument_id: String
    side: OrderSide
    order_type: OrderType
    quantity: Float
    filled_quantity: Float
    remaining_quantity: Float
    price: Optional<Float>
    average_fill_price: Optional<Float>
    status: OrderStatus
    created_at: DateTime
    updated_at: DateTime
    fills: List<Fill>
}

struct Fill {
    fill_id: String
    order_id: String
    quantity: Float
    price: Float
    timestamp: DateTime
    venue: String
    commission: Float
}

struct ExecutionInstructions {
    max_participation_rate: Optional<Float>
    min_fill_size: Optional<Float>
    execution_style: String
    urgency: String
}

// REST API Endpoints
POST /api/v1/orders
    Request: OrderCreationRequest
    Response: Order

GET /api/v1/orders/{order_id}
    Response: Order

PUT /api/v1/orders/{order_id}/cancel
    Response: Order

GET /api/v1/orders/{order_id}/fills
    Response: List<Fill>

POST /api/v1/orders/{order_id}/modify
    Request: OrderModificationRequest
    Response: Order
```

### Event Output
```pseudo
Event order_created {
    event_id: String
    timestamp: DateTime
    order_created: OrderCreatedData
}

struct OrderCreatedData {
    order_id: String
    instrument_id: String
    side: String
    order_type: String
    quantity: Float
    price: Float
    status: String
    execution_instructions: ExecutionInstructionsData
}

struct ExecutionInstructionsData {
    max_participation_rate: Float
    execution_style: String
    urgency: String
}

// Example Event Data
{
    event_id: "uuid",
    timestamp: "2025-06-21T10:00:00.000Z",
    order_created: {
        order_id: "order_20250621_001",
        instrument_id: "AAPL",
        side: "BUY",
        order_type: "LIMIT",
        quantity: 100,
        price: 150.25,
        status: "PENDING",
        execution_instructions: {
            max_participation_rate: 0.10,
            execution_style: "PASSIVE",
            urgency: "NORMAL"
        }
    }
}
```

## Database Schema

### PostgreSQL (Event Sourcing)
```pseudo
Table order_events {
    id: UUID (primary key, auto-generated)
    order_id: String (required, max_length: 100)
    event_type: String (required, max_length: 50)
    event_data: JSON (required)
    event_timestamp: Timestamp (default: now)
    sequence_number: Integer (auto-increment)
    created_at: Timestamp (default: now)
}

Table order_snapshots {
    order_id: String (primary key, max_length: 100)
    order_data: JSON (required)
    version: Integer (required)
    snapshot_timestamp: Timestamp (default: now)
}

Table order_fills {
    id: UUID (primary key, auto-generated)
    fill_id: String (required, unique, max_length: 100)
    order_id: String (required, max_length: 100)
    quantity: Decimal (required, precision: 15, scale: 6)
    price: Decimal (required, precision: 15, scale: 6)
    venue: String (max_length: 100)
    commission: Decimal (precision: 10, scale: 4)
    fill_timestamp: Timestamp (default: now)
    created_at: Timestamp (default: now)
}
```

### Redis Caching
```pseudo
Cache order_cache {
    // Active orders
    "order:{order_id}": Order (TTL: 24h)

    // Order status
    "order_status:{order_id}": OrderStatus (TTL: 1h)

    // Order fills
    "order_fills:{order_id}": List<Fill> (TTL: 24h)

    // Order queue
    "order_queue:{priority}": List<String> (TTL: 30m)
}
```

## Implementation Estimation

### Priority: **CRITICAL** (Core execution functionality)
### Estimated Time: **6-7 weeks**

#### Week 1-2: Core Order Management
- Java service setup with Spring Boot
- Event sourcing implementation with Axon
- Basic order lifecycle management
- **Effort**: 2 developers × 2 weeks = 4 dev-weeks

#### Week 3-4: Advanced Order Features
- Order modification and cancellation
- Fill tracking and aggregation
- Order state management
- **Effort**: 2 developers × 2 weeks = 4 dev-weeks

#### Week 5-6: Event Sourcing & Performance
- Event sourcing optimization
- Order snapshots and replay
- Performance optimization
- **Effort**: 2 developers × 2 weeks = 4 dev-weeks

#### Week 7: Integration & Testing
- Integration with execution services
- Performance testing and optimization
- Audit trail validation
- **Effort**: 2 developers × 1 week = 2 dev-weeks

### Total Effort: **14 dev-weeks**
### Team Size: **2 developers** (1 senior Java developer, 1 mid-level developer)
### Dependencies: Market data, broker integration, execution services

### Success Criteria:
- P99 order processing < 50ms
- 99.99% order integrity
- Complete audit trail for all orders
- Support for 10K+ orders per day
- Zero order data loss
