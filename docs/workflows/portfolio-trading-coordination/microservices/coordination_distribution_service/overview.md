# Coordination Distribution Service

## Responsibility
Event streaming and API management for distributing coordinated trading decisions to downstream workflows. Manages topic routing, subscription management, and provides both streaming and REST APIs for coordination access.

## Technology Stack
- **Language**: Go + Apache Pulsar + gRPC
- **Protocols**: Apache Pulsar, gRPC, WebSocket, REST
- **Data Processing**: Polars for high-performance data manipulation (5-10x faster than pandas)
- **Analytics**: DuckDB for complex analytical queries and aggregations
- **ML Framework**: JAX for mathematical optimization and advanced models
- **Scaling**: Horizontal by topic partitions and consumer groups
- **NFRs**: P99 distribution latency < 20ms, 99.99% delivery guarantee, 200K+ events/sec

## API Specification

### Streaming APIs
```pseudo
// Data Models
struct CoordinationDistribution {
    event_id: String
    timestamp: DateTime
    data_type: String  // "decision", "coordination", "policy"
    data: Any
    routing: RoutingInfo
}

struct RoutingInfo {
    topic: String
    routing_key: String
    target_workflows: List<String>
}

struct DecisionsResponse {
    symbol: String
    decisions: List<CoordinatedTradingDecision>
    timestamp: DateTime
}

// gRPC Service Interface
interface CoordinationStream {
    method streamCoordination(request: StreamRequest) -> Stream<CoordinationMessage>
    method manageSubscription(request: SubscriptionRequest) -> SubscriptionResponse
}

// REST API Endpoints
GET /api/v1/coordination/decisions/{symbol}
    Response: DecisionsResponse
```

### Event Output
```pseudo
Event coordination_distributed {
    event_id: String
    timestamp: DateTime
    distribution: DistributionInfo
    data: CoordinationData
}

struct DistributionInfo {
    topic: String
    routing_key: String
    target_workflows: List<String>
}

struct CoordinationData {
    decision_id: String
    instrument_id: String
    action: String
    quantity: Integer
    confidence: Float
    coordination_quality: Float
}

// Example Event Data
{
    event_id: "uuid",
    timestamp: "2025-06-21T10:00:00.000Z",
    distribution: {
        topic: "coordination/decisions/processed",
        routing_key: "AAPL.decision.buy",
        target_workflows: ["portfolio_management", "trade_execution"]
    },
    data: {
        decision_id: "decision_001",
        instrument_id: "AAPL",
        action: "buy",
        quantity: 100,
        confidence: 0.85,
        coordination_quality: 0.88
    }
}
```

## Database Schema

### PostgreSQL (Command Side)
```pseudo
Table coordination_topics {
    id: UUID (primary key, auto-generated)
    topic_name: String (required, unique, max_length: 200)
    data_type: String (required, max_length: 50)
    partitions: Integer (required, default: 1)
    enabled: Boolean (default: true)
    created_at: Timestamp (default: now)
}

Table coordination_subscriptions {
    id: UUID (primary key, auto-generated)
    subscription_id: String (required, unique, max_length: 100)
    topic_name: String (required, max_length: 200)
    subscriber_workflow: String (required, max_length: 100)
    filters: JSON
    status: String (default: 'active', max_length: 20)
    created_at: Timestamp (default: now)
}
```

## Implementation Estimation

### Priority: **HIGH** (Critical distribution layer)
### Estimated Time: **2-3 weeks**

#### Week 1-2: Core Distribution Engine
- Go service setup with Pulsar integration
- Topic management and subscription handling
- Message routing and distribution
- **Effort**: 2 developers × 2 weeks = 4 dev-weeks

#### Week 3: Integration & Testing
- gRPC and REST API implementation
- Integration with all coordination services
- Performance testing and optimization
- **Effort**: 2 developers × 1 week = 2 dev-weeks

### Total Effort: **6 dev-weeks**
### Team Size: **2 developers** (1 senior Go developer, 1 mid-level developer)
### Dependencies: Apache Pulsar cluster, coordination processing services

### Success Criteria:
- Distribute 200K+ events per second
- P99 distribution latency < 20ms
- 99.99% message delivery guarantee
- Support 200+ concurrent subscribers
- Zero message loss during normal operations
