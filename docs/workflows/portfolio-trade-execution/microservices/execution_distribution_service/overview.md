# Execution Distribution Service

## Responsibility
Event streaming and API management for distributing trade execution data to downstream workflows. Manages topic routing, subscription management, and provides both streaming and REST APIs for execution access.

## Technology Stack
- **Language**: Go + Apache Kafka + gRPC for ultra-low latency distribution
- **Protocols**: Apache Kafka, gRPC, WebSocket, REST
- **Data Processing**: Polars for high-performance data manipulation (5-10x faster than pandas)
- **Analytics**: DuckDB for complex analytical queries and aggregations
- **ML Framework**: JAX for custom optimization algorithms and advanced models
- **Scaling**: Horizontal by topic partitions and consumer groups
- **NFRs**: P99 distribution latency < 10ms, 99.99% delivery guarantee, 1M+ events/sec

## API Specification

### Streaming APIs
```pseudo
// Enumerations
enum ExecutionDataType {
    ORDER_UPDATE,
    FILL_NOTIFICATION,
    EXECUTION_REPORT,
    SETTLEMENT_UPDATE,
    TCA_RESULT,
    VENUE_STATUS
}

// Data Models
struct ExecutionDistribution {
    event_id: String
    timestamp: DateTime
    data_type: ExecutionDataType
    data: Any
    routing: RoutingInfo
    metadata: ExecutionMetadata
}

struct RoutingInfo {
    topic: String
    routing_key: String
    target_workflows: List<String>
    priority: Integer
    delivery_guarantee: String
}

struct ExecutionMetadata {
    source_service: String
    execution_venue: String
    latency_sensitive: Boolean
    audit_required: Boolean
    compliance_flags: List<String>
}

// gRPC Service Interface
interface ExecutionStream {
    method streamExecution(request: StreamRequest) -> Stream<ExecutionMessage>
    method manageSubscription(request: SubscriptionRequest) -> SubscriptionResponse
}

// REST API Endpoints
GET /api/v1/execution/{order_id}/status
    Response: OrderExecutionStatus

GET /api/v1/execution/fills/{trade_id}
    Response: FillDetails

GET /api/v1/execution/reports/{portfolio_id}
    Parameters: start_date, end_date
    Response: ExecutionReport

WebSocket /api/v1/execution/stream
    Parameters: order_ids, data_types
    Response: Stream<ExecutionUpdate>
```

### Event Output
```pseudo
Event execution_distributed {
    event_id: String
    timestamp: DateTime
    distribution: DistributionInfo
    data: ExecutionData
    metadata: ExecutionMetadata
}

struct DistributionInfo {
    topic: String
    routing_key: String
    target_workflows: List<String>
    priority: Integer
    delivery_guarantee: String
}

struct ExecutionData {
    order_id: String
    fill_id: String
    instrument_id: String
    quantity: Float
    price: Float
    venue: String
    execution_time: DateTime
    commission: Float
}

struct ExecutionMetadata {
    source_service: String
    execution_venue: String
    latency_sensitive: Boolean
    audit_required: Boolean
    compliance_flags: List<String>
}

// Example Event Data
{
    event_id: "uuid",
    timestamp: "2025-06-21T10:00:00.000Z",
    distribution: {
        topic: "execution/fills/processed",
        routing_key: "AAPL.fill.complete",
        target_workflows: ["portfolio_management", "reporting_analytics"],
        priority: 1,
        delivery_guarantee: "at_least_once"
    },
    data: {
        order_id: "order_20250621_001",
        fill_id: "fill_20250621_001",
        instrument_id: "AAPL",
        quantity: 100,
        price: 150.25,
        venue: "NYSE",
        execution_time: "2025-06-21T10:00:00.000Z",
        commission: 1.50
    },
    metadata: {
        source_service: "order_management_service",
        execution_venue: "NYSE",
        latency_sensitive: true,
        audit_required: true,
        compliance_flags: ["MiFID_II", "RegNMS"]
    }
}
```

## Database Schema

### PostgreSQL (Command Side)
```pseudo
Table execution_topics {
    id: UUID (primary key, auto-generated)
    topic_name: String (required, unique, max_length: 200)
    data_type: String (required, max_length: 50)
    partitions: Integer (required, default: 1)
    retention_policy: String (default: '7d', max_length: 50)
    latency_requirement_ms: Integer (default: 100)
    enabled: Boolean (default: true)
    created_at: Timestamp (default: now)
}

Table execution_subscriptions {
    id: UUID (primary key, auto-generated)
    subscription_id: String (required, unique, max_length: 100)
    topic_name: String (required, max_length: 200)
    subscriber_workflow: String (required, max_length: 100)
    filters: JSON
    delivery_guarantee: String (default: 'at_least_once', max_length: 20)
    status: String (default: 'active', max_length: 20)
    created_at: Timestamp (default: now)
}

Table distribution_audit {
    id: UUID (primary key, auto-generated)
    event_id: String (required, max_length: 100)
    topic_name: String (required, max_length: 200)
    routing_key: String (required, max_length: 200)
    delivery_status: String (required, max_length: 20)
    delivery_time_ms: Float
    retry_count: Integer (default: 0)
    created_at: Timestamp (default: now)
}
```

### Redis Caching
```pseudo
Cache execution_distribution_cache {
    // Latest execution status
    "exec_status:{order_id}": OrderStatus (TTL: 1h)

    // Fill cache
    "fills:{order_id}": List<Fill> (TTL: 24h)

    // Topic stats
    "topic_stats:{topic}": TopicStats (TTL: 5m)

    // Subscription status
    "subscription:{id}": SubscriptionStatus (TTL: 30m)
}
```

## Implementation Estimation

### Priority: **HIGH** (Critical distribution layer)
### Estimated Time: **3-4 weeks**

#### Week 1-2: Core Distribution Engine
- Go service setup with Kafka integration
- Ultra-low latency message routing
- Topic management and subscription handling
- **Effort**: 2 developers × 2 weeks = 4 dev-weeks

#### Week 2-3: API Development
- gRPC streaming API implementation
- REST API for execution access
- WebSocket support for real-time clients
- **Effort**: 1 developer × 2 weeks = 2 dev-weeks

#### Week 4: Integration & Testing
- Integration with all execution services
- Latency optimization and performance testing
- Audit trail and compliance features
- **Effort**: 2 developers × 1 week = 2 dev-weeks

### Total Effort: **8 dev-weeks**
### Team Size: **2 developers** (1 senior Go developer, 1 mid-level developer)
### Dependencies: Apache Kafka cluster, execution processing services

### Success Criteria:
- Distribute 1M+ events per second
- P99 distribution latency < 10ms
- 99.99% message delivery guarantee
- Support 1000+ concurrent subscribers
- Zero message loss during normal operations
- Complete audit trail for compliance
