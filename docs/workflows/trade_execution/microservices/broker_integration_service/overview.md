# Broker Integration Service

## Responsibility
Multi-broker connectivity and order routing with FIX protocol support. Manages connections to multiple brokers, handles order routing decisions, and provides unified interface for trade execution across different venues.

## Technology Stack
- **Language**: Java + QuickFIX/J + Spring Boot
- **Libraries**: QuickFIX/J, Spring Boot, Apache Camel for routing
- **Scaling**: Horizontal by broker connections, connection pooling
- **NFRs**: P99 order routing < 30ms, 99.9% connection uptime, FIX protocol compliance

## API Specification

### Core APIs
```pseudo
// Enumerations
enum BrokerType {
    PRIME_BROKER,
    EXECUTION_BROKER,
    DARK_POOL,
    ECN,
    MARKET_MAKER
}

enum ConnectionStatus {
    CONNECTED,
    DISCONNECTED,
    CONNECTING,
    ERROR,
    MAINTENANCE
}

enum RoutingStrategy {
    BEST_PRICE,
    FASTEST_EXECUTION,
    LOWEST_COST,
    SMART_ORDER_ROUTING,
    MANUAL
}

// Data Models
struct BrokerConnection {
    broker_id: String
    broker_name: String
    broker_type: BrokerType
    connection_status: ConnectionStatus
    fix_session_id: String
    capabilities: List<String>
    last_heartbeat: DateTime
}

struct OrderRoutingRequest {
    order_id: String
    instrument_id: String
    side: OrderSide
    quantity: Float
    order_type: String
    routing_strategy: RoutingStrategy
    venue_preferences: List<String>
}

struct OrderRoutingResponse {
    order_id: String
    selected_broker: String
    routing_rationale: String
    estimated_execution_time: Integer
    estimated_cost: Float
    fix_message_sent: Boolean
}

struct BrokerCapabilities {
    broker_id: String
    supported_instruments: List<String>
    supported_order_types: List<String>
    execution_quality_metrics: Map<String, Float>
    cost_structure: Map<String, Float>
}

// REST API Endpoints
POST /api/v1/brokers/route
    Request: OrderRoutingRequest
    Response: OrderRoutingResponse

GET /api/v1/brokers/status
    Response: List<BrokerConnection>

GET /api/v1/brokers/{broker_id}/capabilities
    Response: BrokerCapabilities

POST /api/v1/brokers/{broker_id}/connect
    Response: ConnectionResult
```

### Event Output
```pseudo
Event order_routed {
    event_id: String
    timestamp: DateTime
    order_routed: OrderRoutedData
}

struct OrderRoutedData {
    order_id: String
    selected_broker: String
    routing_strategy: String
    routing_rationale: String
    estimated_execution_time: Integer
    estimated_cost: Float
    fix_message_sent: Boolean
    fix_session_id: String
}

// Example Event Data
{
    event_id: "uuid",
    timestamp: "2025-06-21T10:00:00.000Z",
    order_routed: {
        order_id: "order_20250621_001",
        selected_broker: "broker_prime_001",
        routing_strategy: "BEST_PRICE",
        routing_rationale: "Best price available with acceptable execution time",
        estimated_execution_time: 250,
        estimated_cost: 0.0015,
        fix_message_sent: true,
        fix_session_id: "SENDER_COMP_ID->TARGET_COMP_ID"
    }
}
```

## Database Schema

### PostgreSQL (Command Side)
```pseudo
Table broker_configurations {
    id: UUID (primary key, auto-generated)
    broker_id: String (required, unique, max_length: 100)
    broker_name: String (required, max_length: 200)
    broker_type: String (required, max_length: 50)
    fix_configuration: JSON (required)
    capabilities: JSON (required)
    cost_structure: JSON
    enabled: Boolean (default: true)
    created_at: Timestamp (default: now)
}

Table broker_connections {
    id: UUID (primary key, auto-generated)
    broker_id: String (required, max_length: 100)
    session_id: String (required, max_length: 200)
    connection_status: String (required, max_length: 20)
    last_heartbeat: Timestamp
    connection_start: Timestamp
    error_count: Integer (default: 0)
    created_at: Timestamp (default: now)
}

Table order_routing_history {
    id: UUID (primary key, auto-generated)
    order_id: String (required, max_length: 100)
    broker_id: String (required, max_length: 100)
    routing_strategy: String (required, max_length: 50)
    routing_decision_time_ms: Float
    execution_quality_score: Float
    actual_cost: Float
    created_at: Timestamp (default: now)
}
```

## Implementation Estimation

### Priority: **CRITICAL** (Essential for trade execution)
### Estimated Time: **5-6 weeks**

#### Week 1-2: Core Broker Integration
- Java service setup with QuickFIX/J
- Basic FIX protocol implementation
- Broker connection management
- **Effort**: 2 developers × 2 weeks = 4 dev-weeks

#### Week 3-4: Order Routing Logic
- Smart order routing algorithms
- Broker selection optimization
- Order routing decision engine
- **Effort**: 2 developers × 2 weeks = 4 dev-weeks

#### Week 5-6: Advanced Features & Integration
- Multi-broker failover mechanisms
- Performance optimization
- Integration testing with brokers
- **Effort**: 2 developers × 2 weeks = 4 dev-weeks

### Total Effort: **12 dev-weeks**
### Team Size: **2 developers** (1 senior Java developer, 1 FIX protocol specialist)
### Dependencies: Broker API access, FIX connectivity, order management

### Success Criteria:
- P99 order routing < 30ms
- 99.9% broker connection uptime
- FIX protocol compliance 100%
- Support for multiple broker types
- Optimal order routing decisions > 90% accuracy
